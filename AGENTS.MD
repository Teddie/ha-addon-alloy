# Repository Guidelines

## Project Overview

This repository publishes a Home Assistant add-on repository for **Grafana Alloy**. The single add-on lives in `alloy/` and ships Home Assistant OS systemd journal logs to a remote Grafana Loki endpoint.

The add-on replaces Promtail for HAOS 11+ systems where older Promtail builds cannot read newer systemd journal formats.

## Repository Layout

- `repository.yaml`: Home Assistant add-on repository metadata.
- `README.md`: top-level installation and usage documentation for users adding this repository to Home Assistant.
- `alloy/config.yaml`: Home Assistant add-on manifest, options schema, ports, supported architectures, permissions, and release metadata.
- `alloy/build.yaml`: base images used by the Home Assistant add-on builder for each architecture.
- `alloy/Dockerfile`: add-on image build. Installs runtime packages and downloads the pinned Grafana Alloy binary.
- `alloy/README.md`: add-on store summary shown in Home Assistant.
- `alloy/DOCS.md`: add-on documentation tab content.
- `alloy/CHANGELOG.md`: release notes for add-on versions.
- `alloy/rootfs/`: files copied into the add-on container image.
- `alloy/rootfs/etc/s6-overlay/s6-rc.d/init-alloy/run`: one-shot init service that reads `/data/options.json` and generates `/etc/alloy/config.alloy`.
- `alloy/rootfs/etc/s6-overlay/s6-rc.d/alloy/run`: long-running s6 service that starts `alloy run`.

## Runtime Flow

1. Home Assistant starts the add-on using the base Debian image from `alloy/build.yaml`.
2. s6 runs `init-alloy` first.
3. `init-alloy` reads `loki_url`, `log_level`, and optional `additional_config` from `/data/options.json`.
4. It detects the available journal directory, preferring `/var/log/journal` and falling back to `/run/log/journal`.
5. It writes the Alloy configuration to `/etc/alloy/config.alloy`.
6. The `alloy` service starts `/usr/bin/alloy run`, listens on `0.0.0.0:12345`, and stores state in `/data/alloy`.

## Development Commands

There is no package manager, unit test suite, or formatter configured in this repository.

Useful local checks:

```sh
git status --short
find alloy/rootfs -type f -exec ls -l {} +
```

Validate YAML syntax if Ruby is available:

```sh
ruby -e 'require "yaml"; %w[repository.yaml alloy/config.yaml alloy/build.yaml].each { |f| YAML.load_file(f); puts "#{f}: ok" }'
```

Check shell script syntax:

```sh
bash -n alloy/rootfs/etc/s6-overlay/s6-rc.d/init-alloy/run
bash -n alloy/rootfs/etc/s6-overlay/s6-rc.d/alloy/run
```

Builds are intended to run through the Home Assistant add-on build flow. If using the upstream builder locally, build the `alloy` add-on for the supported architectures declared in `alloy/config.yaml` and `alloy/build.yaml`: `amd64` and `aarch64`.

## Editing Guidelines

- Keep the add-on manifest, docs, and changelog synchronized when changing user-visible behavior or options.
- Keep `alloy/config.yaml` option defaults aligned with the values handled in `init-alloy/run`.
- Preserve executable permissions on s6 `run` scripts.
- Avoid introducing dependencies unless they are installed in `alloy/Dockerfile` or already provided by the Home Assistant base image.
- The generated Alloy config is created at container startup. Do not add a checked-in generated `config.alloy`; update `init-alloy/run` instead.
- Treat `additional_config` as raw Alloy syntax appended as-is. Do not assume it is valid or safe to parse with shell string operations.
- Use POSIX-friendly shell where practical, but the runtime scripts currently execute with `bash` via `#!/usr/bin/with-contenv bash`.
- Keep comments concise and focused on non-obvious behavior, especially around journal path detection and Alloy relabeling.

## Release Checklist

When preparing a release:

1. Update `version` in `alloy/config.yaml`.
2. Update `alloy/CHANGELOG.md` with the release date and user-visible changes.
3. If the bundled Alloy version changes, update `ALLOY_VERSION` in `alloy/Dockerfile` and mention it in the changelog.
4. Confirm docs in `README.md`, `alloy/README.md`, and `alloy/DOCS.md` still match the available options and behavior.
5. Run the YAML and shell syntax checks listed above.

## Home Assistant Add-on Notes

- `journald: true` in `alloy/config.yaml` is required for journal access.
- Port `12345/tcp` exposes the Alloy debug UI and is also used by the watchdog readiness check at `/-/ready`.
- `apparmor: false` is currently set; changes to confinement or journal access should be tested on a real HAOS installation.
- Supported architectures are `amd64` and `aarch64`; keep `alloy/config.yaml`, `alloy/build.yaml`, and `alloy/Dockerfile` architecture handling in sync.

## Agent Conduct

- Before editing, inspect the relevant files and check `git status --short`.
- Do not revert unrelated user changes.
- Prefer small, targeted patches over broad refactors.
- If a command needs network access, elevated permissions, or a Home Assistant builder that is not installed locally, say so explicitly instead of masking the limitation.
