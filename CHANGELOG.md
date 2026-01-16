# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-01-16

### Added

- **Self-healing update system** with automatic retry and targeted fixes
  - Download verification (size > 150MB)
  - Extraction verification (V8 snapshots, main binary)
  - Version verification (package.json check)
  - Startup health check (10s stability test)

- **Automatic rollback** when all recovery attempts fail
  - Keeps last 3 backups in `/opt/beeper-backups/`
  - Restores previous working version automatically

- **Pre-flight system checks**
  - Network connectivity
  - Disk space (500MB minimum)
  - Required permissions
  - Running Beeper detection

- **`beeper-version` script** for quick status check
  - Shows installed, latest, and AUR versions
  - Indicates if updates are available
  - Shows how far behind AUR is

- **CLI options**
  - `--check` / `-c`: Check without installing
  - `--changelog` / `-l`: Open changelog in browser
  - `--force` / `-f`: Force reinstall
  - `--notify` / `-n`: Desktop notifications
  - `--help` / `-h`: Show help

- **Systemd integration**
  - Timer and service files for automatic updates
  - Both system-wide and user service variants

- **GitHub Actions** for shellcheck linting

### Why This Exists

Beeper's built-in updater doesn't work on Arch Linux when installed from AUR.
The app downloads updates but can't overwrite pacman-managed files. This script
downloads directly from Beeper's API, bypassing both the broken built-in updater
and the often-outdated AUR package.

[1.0.0]: https://github.com/robertogogoni/update-beeper/releases/tag/v1.0.0
