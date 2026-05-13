# Changelog

## 1.2.2 - 2026-05-13

### Fixed
- Default logs to `level="info"` instead of journal priority, then override when a recognizable message-level value is present
- Extract levels from s6-style service messages such as `s6-rc: info: ...`

## 1.2.1 - 2026-05-13

### Fixed
- Prefer embedded `level=` fields over the journal priority when labeling structured log lines

## 1.2.0 - 2026-05-13

### Changed
- Updated Grafana Alloy to v1.16.1

## 1.1.0 - 2026-05-13

### Changed
- Updated Grafana Alloy to v1.14.1

### Fixed
- Extract Home Assistant log levels from message text so container stderr entries are not all labeled as `error`

## 1.0.0 - 2026-02-21

### Added
- Initial release
- Grafana Alloy v1.13.1
- Systemd journal log shipping to Loki
- Journal field relabeling (unit, hostname, syslog_identifier, transport, container_name, level)
- Debug UI on port 12345
- Configurable Loki URL, log level, and additional config
- Watchdog health check via Alloy's `/-/ready` endpoint
- Support for amd64 and aarch64 architectures
