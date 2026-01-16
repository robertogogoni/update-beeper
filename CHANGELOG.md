# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- **Wayland native rendering support** for Hyprland, Sway, and other Wayland compositors
  - Automatically detects Wayland via `$WAYLAND_DISPLAY`
  - Tests Beeper startup with Ozone platform flags (`--enable-features=UseOzonePlatform --ozone-platform=wayland`)
  - Creates desktop file override at `~/.local/share/applications/beeper.desktop`
  - Fixes blank/white window issues on Wayland without XWayland

- **Sleep/wake stability fix** via `--disable-gpu-compositing` flag
  - Prevents blank screen after screensaver, screen lock, or system sleep
  - Included in Wayland flags by default

- **System compatibility checks** run automatically before updates
  - Architecture verification (x86_64 required)
  - Distro detection (warns if not Arch-based, still works)
  - Dependency check (curl, sudo required)
  - Optional dependency warning (notify-send)

### Fixed

- **Directory leak in `backup_current()`** - Now uses subshell to prevent working directory changes from affecting the main script
- **Directory leak in extraction loop** - Extraction now runs in isolated subshell
- **UX: "Configuring Wayland" message** - Only displays when actually on Wayland
- Removed hardcoded username/UID from systemd service file
- Scripts now work for any user without modification

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
