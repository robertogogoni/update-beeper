# update-beeper

A self-healing Beeper Desktop updater for Linux (Arch-focused).

## Features

- **Self-healing updates** - Automatically retries failed downloads/extractions with fixes
- **Automatic rollback** - Restores previous version if update fails after all retries
- **Health checks** - Verifies Beeper starts and runs stable after update
- **Pre-flight validation** - Checks permissions, disk space, network before updating
- **Smart version detection** - Compares installed vs latest from Beeper API
- **AUR awareness** - Notes when AUR package is behind direct releases

## Installation

```bash
# Download the script
curl -o ~/.local/bin/update-beeper https://raw.githubusercontent.com/robertogogoni/update-beeper/main/update-beeper
chmod +x ~/.local/bin/update-beeper

# Or clone the repo
git clone https://github.com/robertogogoni/update-beeper.git
cp update-beeper/update-beeper ~/.local/bin/
```

Make sure `~/.local/bin` is in your PATH.

## Usage

```bash
update-beeper              # Check and install updates
update-beeper --check      # Check only, don't install
update-beeper --changelog  # Open changelog for installed version
update-beeper --force      # Force reinstall even if up to date
update-beeper --notify     # Send desktop notification (for cron/timers)
```

### Options

| Option | Short | Description |
|--------|-------|-------------|
| `--check` | `-c` | Check for updates without installing |
| `--changelog` | `-l` | Show changelog for installed version |
| `--notify` | `-n` | Send desktop notification (for cron/timer use) |
| `--force` | `-f` | Force update even if already on latest |
| `--help` | `-h` | Show help message |

## How It Works

```
Download → Verify size (>150MB) → Extract → Verify critical files → Install → Verify version → Test startup
    ↓           ↓                    ↓              ↓                  ↓           ↓              ↓
  Retry ←── Fix: re-download ←── Fix: clear ←── Fix: clear ←── Fix: perms ←── Fix: perms ←── Fix: cache
    ↓                              temp dir       extraction
    └──────────────────────── ALL FAILED? → Automatic Rollback
```

### Critical Files Verified

- `beepertexts` - Main binary
- `snapshot_blob.bin` - V8 JavaScript snapshot
- `v8_context_snapshot.bin` - V8 context snapshot
- `resources/app/package.json` - Version source of truth

### Self-Healing Actions

| Failure | Fix Applied |
|---------|-------------|
| Download too small | Clear temp, wait, retry |
| Missing critical files | Clear extraction dir, retry |
| Wrong version installed | Clear temp, fresh download |
| Startup crash | Clear Electron cache, retry |
| All retries exhausted | Automatic rollback to backup |

## Requirements

- Linux (tested on Arch)
- `curl`, `sudo`, `notify-send` (optional)
- Beeper Desktop installed in `/opt/beeper`

## File Locations

| Path | Purpose |
|------|---------|
| `/opt/beeper/` | Installation directory |
| `/opt/beeper-backups/` | Rolling backups (last 3) |
| `~/.config/BeeperTexts/` | User config + caches |

## License

MIT
