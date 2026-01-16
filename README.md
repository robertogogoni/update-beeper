# update-beeper

```
    ╔══════════════════════════════════════════════════════════════════════════╗
    ║                                                                          ║
    ║   ██████╗ ███████╗███████╗██████╗ ███████╗██████╗                        ║
    ║   ██╔══██╗██╔════╝██╔════╝██╔══██╗██╔════╝██╔══██╗                       ║
    ║   ██████╔╝█████╗  █████╗  ██████╔╝█████╗  ██████╔╝                       ║
    ║   ██╔══██╗██╔══╝  ██╔══╝  ██╔═══╝ ██╔══╝  ██╔══██╗                       ║
    ║   ██████╔╝███████╗███████╗██║     ███████╗██║  ██║                       ║
    ║   ╚═════╝ ╚══════╝╚══════╝╚═╝     ╚══════╝╚═╝  ╚═╝                       ║
    ║                                                                          ║
    ║   ██╗   ██╗██████╗ ██████╗  █████╗ ████████╗███████╗██████╗              ║
    ║   ██║   ██║██╔══██╗██╔══██╗██╔══██╗╚══██╔══╝██╔════╝██╔══██╗             ║
    ║   ██║   ██║██████╔╝██║  ██║███████║   ██║   █████╗  ██████╔╝             ║
    ║   ██║   ██║██╔═══╝ ██║  ██║██╔══██║   ██║   ██╔══╝  ██╔══██╗             ║
    ║   ╚██████╔╝██║     ██████╔╝██║  ██║   ██║   ███████╗██║  ██║             ║
    ║    ╚═════╝ ╚═╝     ╚═════╝ ╚═╝  ╚═╝   ╚═╝   ╚══════╝╚═╝  ╚═╝             ║
    ║                                                                          ║
    ║                🐝  Self-healing • Auto-rollback • Arch Linux  🐝         ║
    ║                                                                          ║
    ╚══════════════════════════════════════════════════════════════════════════╝
```

[![Lint](https://github.com/robertogogoni/update-beeper/actions/workflows/lint.yml/badge.svg)](https://github.com/robertogogoni/update-beeper/actions/workflows/lint.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A self-healing Beeper Desktop updater for Linux, built specifically for Arch Linux users.

## Why This Exists

### The Problem: Beeper's Built-in Updater Doesn't Work on Arch

If you've installed Beeper from the AUR (`beeper-v4-bin`), you've probably noticed that clicking **"Update Available"** inside Beeper does... nothing. The app downloads the update, prompts you to restart, and then you're still on the old version.

**Why?** Beeper's built-in updater is designed for standalone AppImage installations where it can replace itself. But on Arch Linux, Beeper is managed by pacman/yay through the AUR package. The app can't overwrite files that pacman owns—so the update silently fails, leaving you stuck on the old version with no error message.

### The Problem: AUR Is Always Behind

Even if you run `yay -Syu beeper-v4-bin`, you're often still behind. The AUR package depends on a maintainer to notice a new release, update the `PKGBUILD`, and push it. This can take hours or even days. Meanwhile, Beeper has already released a newer version that you can't get.

**This script solves both problems:**
- ✅ Downloads directly from Beeper's API (always the latest)
- ✅ Installs immediately without waiting for AUR
- ✅ Tells you when AUR catches up so you can resync

### Bonus: Self-Healing After a Real Failure

During development, a routine update broke everything—the AppImage extraction silently failed, leaving critical V8 snapshot files missing:

```
FATAL:gin/v8_initializer.cc:705] Error loading V8 startup snapshot file
```

Beeper crashed on startup. No warning during install. No automatic recovery. This script now verifies every critical file and automatically rolls back if anything goes wrong.

## How Beeper Updates Work (And Why They Break)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    BEEPER'S BUILT-IN UPDATE (Broken on Arch)                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   1. Beeper detects new version via API                                     │
│   2. Downloads update in background                                         │
│   3. Prompts: "Restart to update"                                           │
│   4. User restarts...                                                       │
│   5. ❌ Update fails silently (pacman owns the files)                       │
│   6. User is still on old version, confused                                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                    THIS SCRIPT (Works on Arch)                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   1. Queries Beeper API for latest version                                  │
│   2. Compares against installed version                                     │
│   3. Downloads AppImage directly from Beeper                                │
│   4. Extracts and verifies all critical files                               │
│   5. Backs up current installation                                          │
│   6. Installs with proper permissions (sudo)                                │
│   7. Verifies Beeper starts and runs stable                                 │
│   8. ✅ You're on the latest version!                                       │
│                                                                             │
│   If anything fails → automatic rollback to backup                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Features

```
  ┌─────────────────────────────────────────────────────────────────┐
  │  🚀 DIRECT DOWNLOAD   Gets latest from Beeper API, skip AUR    │
  │  🔄 SELF-HEALING      Retries with targeted fixes              │
  │  ⏪ AUTO-ROLLBACK     Restores previous version on failure     │
  │  🏥 HEALTH CHECKS     Verifies Beeper runs stable (10s)        │
  │  🛫 PRE-FLIGHT        Validates permissions, space, network    │
  │  📦 AUR AWARE         Tells you when AUR catches up            │
  │  📊 VERSION STATUS    Quick check with beeper-version          │
  │  ⏰ AUTO UPDATES      Systemd timer for set-and-forget         │
  └─────────────────────────────────────────────────────────────────┘
```

## Installation

### Quick Install (Recommended)

```bash
curl -fsSL https://raw.githubusercontent.com/robertogogoni/update-beeper/main/install.sh | bash
```

### Manual Install

```bash
# Download scripts
curl -o ~/.local/bin/update-beeper \
  https://raw.githubusercontent.com/robertogogoni/update-beeper/main/update-beeper
curl -o ~/.local/bin/beeper-version \
  https://raw.githubusercontent.com/robertogogoni/update-beeper/main/beeper-version

# Make executable
chmod +x ~/.local/bin/update-beeper ~/.local/bin/beeper-version
```

### Clone & Install

```bash
git clone https://github.com/robertogogoni/update-beeper.git
cd update-beeper
./install.sh
```

Make sure `~/.local/bin` is in your PATH.

## Usage

### Check Version Status

```bash
beeper-version
```

```
🐝 Beeper Version Status
─────────────────────────────────────

  Installed:  4.2.482
  Latest:     4.2.482
  AUR:        4.2.455

  ✓ You're on the latest version!
  ℹ AUR is behind by ~27 releases
```

### Update Beeper

```bash
update-beeper              # Check and install updates
update-beeper --check      # Check only, don't install
update-beeper --changelog  # Open changelog for installed version
update-beeper --force      # Force reinstall even if up to date
update-beeper --notify     # Send desktop notification (for automation)
```

### Options

| Option | Short | Description |
|--------|-------|-------------|
| `--check` | `-c` | Check for updates without installing |
| `--changelog` | `-l` | Show changelog for installed version |
| `--notify` | `-n` | Send desktop notification (for cron/timer use) |
| `--force` | `-f` | Force update even if already on latest |
| `--help` | `-h` | Show help message |

## Automatic Updates (Systemd)

Set up automatic daily update checks with desktop notifications:

```bash
# Copy systemd user files
mkdir -p ~/.config/systemd/user
curl -fsSL https://raw.githubusercontent.com/robertogogoni/update-beeper/main/systemd/update-beeper-user.service \
  -o ~/.config/systemd/user/update-beeper.service
curl -fsSL https://raw.githubusercontent.com/robertogogoni/update-beeper/main/systemd/update-beeper-user.timer \
  -o ~/.config/systemd/user/update-beeper.timer

# Enable the timer
systemctl --user daemon-reload
systemctl --user enable --now update-beeper.timer

# Check timer status
systemctl --user list-timers update-beeper.timer
```

The timer runs daily between 10:00-14:00 (randomized to avoid hammering Beeper's servers).

## Self-Healing Pipeline

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

- Arch Linux (or Arch-based distros)
- `curl`, `sudo`
- `notify-send` (optional, for notifications)
- Beeper Desktop (from AUR or manual install in `/opt/beeper`)

## File Locations

```
/opt/beeper/              ← Installation directory
/opt/beeper-backups/      ← Rolling backups (last 3 versions)
~/.config/BeeperTexts/    ← User config + Electron caches
/tmp/beeper-update/       ← Temporary download/extract dir
```

## FAQ

**Q: Should I still use the AUR package?**

Yes! This script works alongside the AUR package. When AUR catches up, you can run `yay -Syu beeper-v4-bin` to resync. The script will tell you when this happens.

**Q: What if an update breaks something?**

The script automatically rolls back to your previous working version. You'll get a notification and can try again later or report the issue.

**Q: Can I automate this?**

Yes! See the [Automatic Updates](#automatic-updates-systemd) section for systemd timer setup.

**Q: How do I check my current version status?**

Run `beeper-version` for a quick overview of installed, latest, and AUR versions.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.

## License

MIT

---

<p align="center">
  <i>Because Beeper's "Update Available" button should actually work</i> 🐝
</p>
