# update-beeper

```
                 ╭──────────────────────────────────────────╮
                 │                                          │
    ┌──────┐     │   ██╗   ██╗██████╗ ██████╗  █████╗ ████████╗███████╗   │
    │ ▓▓▓▓ │     │   ██║   ██║██╔══██╗██╔══██╗██╔══██╗╚══██╔══╝██╔════╝   │
    │ ▓▓▓▓ │ ──► │   ██║   ██║██████╔╝██║  ██║███████║   ██║   █████╗     │
    │ ▓▓▓▓ │     │   ██║   ██║██╔═══╝ ██║  ██║██╔══██║   ██║   ██╔══╝     │
    └──────┘     │   ╚██████╔╝██║     ██████╔╝██║  ██║   ██║   ███████╗   │
    Beeper       │    ╚═════╝ ╚═╝     ╚═════╝ ╚═╝  ╚═╝   ╚═╝   ╚══════╝   │
    AppImage     │                                                        │
                 │           🐝  BEEPER DESKTOP UPDATER  🐝               │
                 │          Self-healing • Auto-rollback                  │
                 ╰──────────────────────────────────────────╯
```

A self-healing Beeper Desktop updater for Linux (Arch-focused).

## Why This Exists

> *"The update broke everything. Beeper wouldn't start. I had to manually restore from backup."*

This script was born from a real failure. During a routine Beeper update, the AppImage extraction silently failed—critical V8 snapshot files were missing. The result?

```
FATAL:gin/v8_initializer.cc:705] Error loading V8 startup snapshot file
```

Beeper crashed on startup. No warning during install. No automatic recovery. Just a broken app and a scramble to restore the backup manually.

**Never again.**

This updater now:
- ✅ Verifies every critical file exists after extraction
- ✅ Tests that Beeper actually starts and stays running
- ✅ Automatically rolls back if anything goes wrong
- ✅ Keeps the last 3 working versions as backups

## Features

```
  ┌─────────────────────────────────────────────────────────────┐
  │  🔄 SELF-HEALING     Retries with targeted fixes            │
  │  ⏪ AUTO-ROLLBACK    Restores previous version on failure   │
  │  🏥 HEALTH CHECKS    Verifies Beeper runs stable (10s)      │
  │  🛫 PRE-FLIGHT       Validates permissions, space, network  │
  │  🔍 SMART DETECT     Compares installed vs Beeper API       │
  │  📦 AUR AWARE        Notes when AUR lags behind releases    │
  └─────────────────────────────────────────────────────────────┘
```

## Installation

```bash
# One-liner install
curl -o ~/.local/bin/update-beeper \
  https://raw.githubusercontent.com/robertogogoni/update-beeper/main/update-beeper \
  && chmod +x ~/.local/bin/update-beeper

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
┌──────────────────────────────────────────────────────────────────────────┐
│                            UPDATE PIPELINE                               │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   📥 Download ──► 📦 Extract ──► 📁 Install ──► 🔍 Verify ──► 🚀 Start  │
│       │              │              │              │              │      │
│       ▼              ▼              ▼              ▼              ▼      │
│   [>150MB?]    [V8 files?]    [Perms OK?]   [Version?]    [Stable?]     │
│       │              │              │              │              │      │
│       │ FAIL         │ FAIL         │ FAIL         │ FAIL         │ FAIL │
│       ▼              ▼              ▼              ▼              ▼      │
│   ┌──────────────────────────────────────────────────────────────┐      │
│   │              🔧 RETRY WITH TARGETED FIX (2x)                 │      │
│   │  • Clear temp dir, re-download                               │      │
│   │  • Clear extraction, retry                                   │      │
│   │  • Fix permissions recursively                               │      │
│   │  • Clear Electron cache                                      │      │
│   └──────────────────────────────────┬───────────────────────────┘      │
│                                      │                                   │
│                                      │ ALL RETRIES EXHAUSTED             │
│                                      ▼                                   │
│   ┌──────────────────────────────────────────────────────────────┐      │
│   │              ⏪ AUTOMATIC ROLLBACK                            │      │
│   │  • Restore from pre-update backup                            │      │
│   │  • Verify rollback works                                     │      │
│   │  • Notify user of failure                                    │      │
│   └──────────────────────────────────────────────────────────────┘      │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

### Critical Files Verified

These files are checked after every extraction—if any are missing, the update retries:

| File | Purpose |
|------|---------|
| `beepertexts` | Main Electron binary |
| `snapshot_blob.bin` | V8 JavaScript snapshot |
| `v8_context_snapshot.bin` | V8 context snapshot |
| `resources/app/package.json` | Version source of truth |

### Self-Healing Actions

| Failure | Diagnosis | Fix Applied |
|---------|-----------|-------------|
| Download too small | Incomplete transfer | Clear temp, wait 2s, retry |
| Missing critical files | Extraction failed | Clear squashfs-root, re-extract |
| Wrong version | Stale download | Clear temp, fresh download |
| Startup crash | Corrupted cache | Clear Electron cache dirs |
| All retries exhausted | Unrecoverable | **Automatic rollback** |

## Requirements

- Linux (tested on Arch)
- `curl`, `sudo`
- `notify-send` (optional, for notifications)
- Beeper Desktop installed in `/opt/beeper`

## File Locations

```
/opt/beeper/              ← Installation directory
/opt/beeper-backups/      ← Rolling backups (last 3 versions)
~/.config/BeeperTexts/    ← User config + Electron caches
/tmp/beeper-update/       ← Temporary download/extract dir
```

## License

MIT

---

<p align="center">
  <i>Built after one too many broken updates</i> 🐝
</p>
