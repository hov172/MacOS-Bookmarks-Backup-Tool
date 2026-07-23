# Bookmark Backup Tool - v2.7.1

![Platform](https://img.shields.io/badge/platform-macOS%2014%2B-blue)
![Swift](https://img.shields.io/badge/Swift-5.9-orange)
![Release](https://img.shields.io/badge/release-v2.7.1-green)
![Tests](https://img.shields.io/badge/tests-63%20passing-brightgreen)

**Requires macOS 14 (Sonoma) or later.**

**Bookmark Backup Tool** is a powerful utility for macOS designed to back up, restore, and manage your web browser bookmarks. It offers both a user-friendly graphical interface (GUI) and a comprehensive command-line interface (CLI) for power users and automation.

Whether you want to perform a quick manual backup, schedule regular automated exports, or migrate bookmarks, this tool has you covered.

**This app is for macOS use. I also built both PowerShell and Windows applications based on a script. These apps\scripts allow users to manage their own bookmarks, and both versions include command-line options for administrators.**

**Other Options**
- 🖥️ **Windows Powershell**: [https://github.com/hov172/PS-Bookmark-Backup-Tool](https://github.com/hov172/PS-Bookmark-Backup-Tool)
- 🖥️ **Windows**: [https://github.com/hov172/Win-Bookmarks-Backup-Tool/tree/main](https://github.com/hov172/Win-Bookmarks-Backup-Tool/tree/main)

**Two variants**: the standard build exposes the Settings tab, while managed deployments can hide it by shipping a `Settings.plist` with `ShowSettingsTab` set to `false`.

<img width="1680" height="961" alt="screenshot" src="https://github.com/user-attachments/assets/d3ca50d6-bd54-4d1e-a5c5-2c408b06ca01" />

## Contents

- [Installation](#installation)
- [Features](#features)
- [Safari & Full Disk Access](#important-note-safari--full-disk-access)
- [Enterprise Deployment (MDM)](#enterprise-deployment-mdm)
- [What's New](#whats-new-in-v271)
- [GUI Usage](#usage-graphical-user-interface-gui)
- [CLI Usage & Command Reference](#usage-command-line-interface-cli)
- [Multi-User Deployment](#multi-user-deployment-scenarios)
- [Building & Releasing](#building--releasing)

## Installation

**Installer (recommended):** download `BookmarkBackupTool-x.y.z.pkg` from the [Releases page](https://github.com/hov172/MacosBookmarksBackup/releases) and run it. It installs **Bookmark Backup Tool** to `/Applications` and creates the `bookmarks-backup` CLI symlink in `/usr/local/bin`.

**Build from source:**
```
git clone https://github.com/hov172/MacosBookmarksBackup.git
cd MacosBookmarksBackup
scripts/create_package.sh        # produces scripts/release/BookmarkBackupTool-x.y.z.pkg
```
See [Building & Releasing](#building--releasing) for signing and notarization.

**Enterprise:** deploy the pkg plus the TCC profile via MDM — see [Enterprise Deployment (MDM)](#enterprise-deployment-mdm).

## Features

- **Dual Interface**: Use the intuitive GUI for easy operation or the robust CLI for scripting.
- **Multi-Browser Support**: Works with Chrome, Edge, Firefox, Safari, and OpenAI ChatGPT Atlas.
- **Profile Management**: Automatically detects and manages multiple user profiles.
- **Flexible Export/Import**:
    - Process any combination of browsers and profiles.
    - Save backups to a custom location, automatically create ZIP archives, or toggle `Zip after export` per run.
- **Versioned Backups & Retention**:
    - Each export is written to a timestamped `Backup_YYYY-MM-DD_HHMMSS` folder (or timestamped ZIP), so older backups are never overwritten.
    - A configurable retention count ("Backups to keep", default 10, 0 = unlimited) prunes the oldest backups automatically.
- **Safe Database Snapshots**:
    - Safari and Firefox databases are exported with SQLite's online backup API, producing consistent single-file snapshots even while the browser is running.
- **Import Validation**:
    - Backup files are validated (JSON structure, SQLite header, plist format) before they overwrite live browser data. Disable per-run with `--no-verify` or via the `verifyIntegrity` setting.
- **Backup Health Monitoring**:
    - Every backup records its outcome; the GUI shows the last run's status, and `check` flags failed or overdue scheduled backups (`check --json` for MDM/fleet monitoring).
- **Content Verification**:
    - Every backup counts its bookmarks (Chrome/Edge/Atlas JSON, Firefox and Safari databases, Safari plist) and ships a `manifest.json` inventory inside the backup; a sharp count drop versus the previous like-for-like run raises a warning before retention prunes the good copies.
- **Scheduled Backups**:
    - Set up automated, recurring backup tasks (daily, weekly, or monthly).
    - Scheduled runs log to `~/Library/Logs/BookmarksBackupTool/`.
- **Configuration Management**:
    - Export and import the app's configuration to a JSON file, making setup on a new machine easy.
- **Health Check & Utilities**:
    - Built-in tools to verify permissions, validate bookmark files, and clean up old logs.

## Supported Browsers

- Google Chrome
- Microsoft Edge
- Mozilla Firefox
- Apple Safari
- OpenAI ChatGPT Atlas

## Important Note: Safari & Full Disk Access

Scheduled backups that include Safari require **Full Disk Access**. Without this permission, automated tasks will fail.

**To grant Full Disk Access:**
1.  Open **System Settings** → **Privacy & Security** → **Full Disk Access**.
2.  Click the `+` button, find **Bookmark Backup Tool**, and add it to the list.
3.  Ensure the toggle next to the app is enabled.

The application will warn you if you create a scheduled task involving Safari without the required permissions.

## Enterprise Deployment (MDM)

For enterprise environments, IT administrators can deploy the included TCC privacy profile (`config/tcc-profile.mobileconfig`) via an MDM solution. This pre-approves Full Disk Access, so end-users do not need to grant it manually. **Note:** TCC profiles can only be installed via MDM, not by a user.

**Important:** a TCC profile matches the app by its code signing requirement, so it only works against a **signed** build:

1. Build and sign a release: `APP_SIGN_IDENTITY="Developer ID Application: …" INSTALLER_SIGN_IDENTITY="Developer ID Installer: …" NOTARY_PROFILE=<profile> scripts/create_package.sh`
2. Regenerate the profile against that exact build: `scripts/generate_tcc_profile.sh "/Applications/Bookmark Backup Tool.app"`
3. Deploy the regenerated `config/tcc-profile.mobileconfig` via MDM.

An unsigned build has no stable code requirement and **cannot** be granted Full Disk Access by any TCC profile.

## Advanced Configuration

The behavior of the Bookmark Backup Tool can be customized for managed environments by deploying a custom `Settings.plist` file. This is ideal for administrators who want to provide a simplified, pre-configured experience for users.

### Hiding the Settings Tab

To prevent users from changing settings, you can hide the "Settings" tab in the GUI.

1.  Create or edit a `Settings.plist` file.
2.  Add the following key-value pair:
```xml
    <key>ShowSettingsTab</key>
    <false/>
```
3.  Deploy this file to `/Library/Application Support/BookmarksBackupTool/Settings.plist` on the target machine.

When the application launches, it will read this system-level setting and hide the "Settings" tab, preventing users from modifying the default configuration.

### What's New in v2.7.1

- **Content verification**: every backup counts its bookmarks and ships a `manifest.json` inventory (per-file browser, count, size, app version, host). The CLI reports "Backed up N bookmarks", the GUI and `check` show the count, and a sharp drop versus the previous like-for-like run raises a warning before retention prunes the good copies.
- **TCC profile regenerated** from the Developer ID-signed build (pins Full Disk Access to the team's code signature).
- **Firefox stub profiles ignored**; SQLite snapshots are single self-contained files.

### What's New in v2.7

- **Backup health monitoring**: every export records its outcome; the GUI shows "Last backup succeeded/FAILED", and `check` reports the last run and flags overdue scheduled backups (no success within twice the scheduled period). `check --json` emits the full health report for MDM extension attributes / fleet monitoring, and `check` exits non-zero when status is not OK.
- **Cross-machine Firefox restore**: imports now match Firefox profiles by their `.default-release`-style suffix (with a single-backup fallback) instead of requiring the random profile-folder prefix to match — migration restores actually restore Firefox now.
- **Firefox stub profiles ignored**: profile folders without a `places.sqlite` (Firefox's empty stubs) are no longer selected for export or preferred as restore targets, fixing "Source file not found" export failures.
- **Self-contained snapshots**: SQLite backup artifacts are single files — WAL/shared-memory scaffolding is cleaned up after the snapshot.
- **Requires macOS 14+**: the GUI migrated to the Observation framework (`@Observable`), eliminating the "Publishing changes from within view updates" warning class; export/import error alerts are now actually presented (previously they were created but never shown).
- **Release automation**: pushing a `vX.Y.Z` tag builds the installer and attaches it to a GitHub Release (unsigned CI build; sign + notarize locally for distribution).

### What's New in v2.6

- **Versioned backups**: exports land in timestamped `Backup_…` folders with automatic retention (`backupRetentionCount` setting, "Backups to keep" in the GUI, `--flat` to opt out).
- **Consistent database snapshots**: Safari/Firefox databases are exported via SQLite's online backup API — safe while the browser is running — with WAL sidecar handling on import.
- **Import safety**: backup files are validated before overwriting live browser data (`verifyIntegrity` setting, `--no-verify` to skip), imports report per-browser success/failure/skip results, and the CLI exits non-zero on failure (`import --json` available for automation).
- **Scheduler fixes**: scheduled jobs now pass a valid `--quiet` flag (previously `--silent`, which every run failed to parse), resolve the executable inside whatever the app bundle is named, and log to `~/Library/Logs/BookmarksBackupTool/` instead of a shared `/tmp` file.
- **Log retention fixed**: log timestamps are ISO-8601 and the retention policy actually prunes old entries now.
- **Release pipeline**: `scripts/create_package.sh` supports Developer ID signing, hardened runtime, and notarization; `scripts/generate_tcc_profile.sh` regenerates the MDM TCC profile from a signed build; CI runs the test suite on every push.

### What's New in v2.5

- **OpenAI Browser**: Added support to export and import Atlas Browser.

### What's New in v2.1

- **Enhanced Settings Tab**: Added a "Choose" button next to the default backup path field, allowing users to easily select a folder location through a file picker dialog.
- **Improved Settings Management**: Fixed an issue where the `ShowSettingsTab` setting wasn't properly loaded from the bundled settings file, ensuring that administrators can reliably hide the settings tab in managed environments.

## Usage: Graphical User Interface (GUI)

Launch the application to access the main window for all primary functions.

- **Main Tab**: Perform immediate exports or imports. Select your browsers, choose a target path, and go.
- **Profiles Tab**: Lists all browser profiles automatically detected on your system.
- **Settings Tab**: Customize default behaviors, such as the default backup path, default browsers, backups to keep, and log retention. You can also manage and view scheduled backups from this tab.

## Usage: Command-Line Interface (CLI)

The CLI is perfect for scripting and automation.

### Command-Line Interface Access

The CLI can be accessed using the full path to the executable inside the application bundle.

#### Direct Path Execution (Always Works)
After installing the application, you can run commands like this:
```
# Full path to the CLI command
/Applications/Bookmark\ Backup\ Tool.app/Contents/MacOS/BookmarksBackupTool --help
```

#### Creating a Symlink for Convenience
The installer creates a `bookmarks-backup` symlink in `/usr/local/bin` automatically. To create it manually:
```
# Create symlink in /usr/local/bin (requires admin password)
sudo ln -s "/Applications/Bookmark Backup Tool.app/Contents/MacOS/BookmarksBackupTool" /usr/local/bin/bookmarks-backup

# Now you can use the shorter command:
bookmarks-backup --version
bookmarks-backup check
```
### CLI Command Reference

*(The following examples assume you have created the `bookmarks-backup` symlink for convenience.)*

#### Basic Commands
```
# Show help information
bookmarks-backup --help

# Show application status and detected profiles
bookmarks-backup status

# List all detected browser profiles
bookmarks-backup list-profiles
```

#### Exporting Bookmarks
```
# Export bookmarks from default browsers to the default location.
# Each run is written to a timestamped Backup_YYYY-MM-DD_HHMMSS subfolder;
# the oldest backups beyond the retention count are pruned automatically.
# Every backup includes a manifest.json inventory and reports its bookmark
# count ("Backed up 531 bookmarks."); a sharp count drop vs the previous
# like-for-like run prints a warning.
bookmarks-backup export

# Export from specific browsers to a custom location
bookmarks-backup export --chrome --firefox --path "/Users/Shared/Backups"

# Export all profiles for all browsers and create a ZIP archive
bookmarks-backup export --all-browsers --all-profiles --zip

# Write files directly into the target path (pre-2.6 layout, no run subfolder)
bookmarks-backup export --flat

# Preview an export without writing files (dry run)
bookmarks-backup export --all-browsers --dry-run
```

#### Importing Bookmarks
```
# Import bookmarks from a backup root: the newest Backup_ folder or ZIP inside
# it is selected automatically. Point at a specific Backup_ folder or ZIP to
# import exactly that run.
bookmarks-backup import --all-browsers --path "/path/to/backup"

# Import from a ZIP archive, forcing the import even if a browser is running
bookmarks-backup import --all-browsers --path "/path/to/backup.zip" --force

# Skip creating a backup of existing bookmarks before import
bookmarks-backup import --all-browsers --path "/path/to/backup" --no-backup

# Skip pre-import validation of the backup files
bookmarks-backup import --all-browsers --path "/path/to/backup" --no-verify

# Machine-readable result (succeeded / failed / skipped / notes); the command
# exits non-zero if any browser failed to import
bookmarks-backup import --all-browsers --path "/path/to/backup" --json
```

#### Scheduled Backups Management
```
# Create a daily scheduled backup at 10:00 AM
bookmarks-backup schedule create --all-browsers --time "10:00" --frequency Daily

# Create a weekly scheduled backup on Mondays at 9:30 AM
# (launchd weekday numbering: 0-7, where 0 and 7 are Sunday, 1 is Monday)
bookmarks-backup schedule create --all-browsers --time "09:30" --frequency Weekly --weekday 1

# Remove the existing scheduled task
bookmarks-backup schedule remove
```

#### Settings Management
```
# Get a specific setting value
bookmarks-backup settings get defaultPath

# Set a new default backup path
bookmarks-backup settings set defaultPath "/Users/Shared/Backups"

# Keep the 20 most recent backups in the target folder (0 = keep all)
bookmarks-backup settings set backupRetentionCount 20

# Export the entire application configuration to a JSON file
bookmarks-backup settings export --output "/path/to/config.json"

# Import an application configuration from a JSON file
bookmarks-backup settings import "/path/to/config.json"
```

#### System Maintenance
```
# Apply the log retention policy and clean up old logs
bookmarks-backup cleanup-logs

# Perform a system health check: permissions, scheduled task, profiles, and
# the last backup's outcome (flags failed or overdue scheduled backups).
# Exits non-zero when the overall status is not OK.
bookmarks-backup check

# Full health report as JSON — ideal for MDM extension attributes
bookmarks-backup check --json
```

## Browser Support
- **Google Chrome** - Supports multiple profiles
- **Microsoft Edge** - Supports multiple profiles
- **Mozilla Firefox** - Supports multiple profiles; empty stub profiles are ignored, and restores match profiles across machines by their `.default-release`-style suffix
- **Apple Safari** - Single profile (requires Full Disk Access for scheduled backups)
- **OpenAI ChatGPT Atlas** - Supports multiple profiles

## Enterprise Deployment Features
- **MDM Deployment**: Can be deployed via Mobile Device Management solutions.
- **TCC Profile**: Privacy profile for automatic Full Disk Access (regenerate per signed build — see Enterprise Deployment above).
- **Silent Operation**: CLI mode supports `--quiet` for automated scripts.
- **JSON Output**: Structured output (`export --json`, `import --json`, `check --json`) for integration with monitoring systems.
- **Configuration Management**: Export/import settings for consistent deployment.
- **Fleet Health**: `check --json` + exit codes make backup health — including bookmark counts and drop warnings — visible to MDM extension attributes.
- **Auditable Backups**: every backup carries a `manifest.json` inventory (files, per-browser bookmark counts, app version, host).

### Common Use Cases

#### Individual User Backup
```
# Daily backup of primary browsers
bookmarks-backup schedule create --chrome --firefox --safari --time "22:00" --frequency Daily
```

#### Enterprise Deployment
```
# Deploy via MDM with TCC profile, then configure organization-wide backup
bookmarks-backup settings set defaultPath "/Company/Backups/Bookmarks"
bookmarks-backup schedule create --chrome --firefox --all-profiles --time "02:00" --frequency Daily
```

#### Migration Scenario
```
# Export all profiles for migration to a new machine
bookmarks-backup export --all-browsers --all-profiles --path "/Migration/Backup" --zip

# Import on the new machine
bookmarks-backup import --all-browsers --path "/Migration/Backup.zip" --force
```

## Usage with TCC Profile

For enterprise environments, this profile **must be deployed via MDM** to automatically grant Full Disk Access permissions.

### Deployment via MDM (Required)
1. Build and sign a release, then regenerate the profile against that build with `scripts/generate_tcc_profile.sh` (the profile embeds the app's code signing requirement and only matches the build it was generated from).
2. Import the regenerated `tcc-profile.mobileconfig` file into your MDM solution.
3. Deploy the profile to target devices along with the signed Bookmark Backup Tool application.
4. Users will have Full Disk Access permissions automatically without manual intervention.

### Manual Installation (Not Supported for TCC)
**Note**: Due to Apple's security policies, manually installing a TCC profile by double-clicking it **will fail** and will not grant the required permissions. This method is blocked by macOS to prevent malicious software from tricking users.

## Multi-User Deployment Scenarios

The Bookmark Backup Tool supports various deployment scenarios for multi-user environments.

#### Individual User Configuration
Each user can configure their own scheduled backups. The `~` symbol automatically expands to the user's home directory.
```
# Each user sets up their own backup to their Desktop
bookmarks-backup schedule create --all-browsers --time "21:00" --frequency Daily --path "~/Desktop/BookmarkBackups"
```

#### System-Wide Scripted Deployment
For automatic configuration of all existing users, an administrator can run a script.
```
#!/bin/bash
# Script to configure scheduled backups for all users

for user_home in /Users/*; do
    username=$(basename "$user_home")
    
    # Skip system users and check if it's a real user directory
    if [[ "$username" != "Shared" && "$username" != "Guest" && -d "$user_home" && -d "$user_home/Desktop" ]]; then
        echo "Setting up backup for user: $username"
        
        # Run as the user to set up their scheduled backup
        sudo -u "$username" /Applications/Bookmark\ Backup\ Tool.app/Contents/MacOS/BookmarksBackupTool schedule create \
            --all-browsers \
            --time "21:00" \
            --frequency Daily \
            --path "~/Desktop/BookmarkBackups" \
            --all-profiles
    fi
done
```

#### Best Practices for Multi-User Deployments
1. **Use TCC Profiles**: Deploy the provided TCC profile via MDM to pre-authorize Full Disk Access.
2. **Standardize Paths**: Use `~` in paths to ensure backups go to each user's personal directories.
3. **Document Procedures**: Create clear instructions for users on setting up their backups.
4. **Monitor Compliance**: Use the `check` command (or `check --json` via MDM) to verify backups are actually succeeding.

## Building & Releasing

- Regenerate the Xcode project after editing `project.yml`: `xcodegen`
- Run the test suite: `swift test`
- Build a signed, notarized installer (see the header of `scripts/create_package.sh` for the required environment variables): `scripts/create_package.sh`
- Regenerate the MDM TCC profile against a signed build: `scripts/generate_tcc_profile.sh "/Applications/Bookmark Backup Tool.app"`
- Pushing a `vX.Y.Z` tag builds an unsigned test installer and attaches it to a GitHub Release.

See [CHANGELOG.md](CHANGELOG.md) for the release history.

## 🌐 Connect With Me
- [GitHub](https://github.com/hov172)  
- [PowerShell Gallery](https://www.powershellgallery.com/profiles/hov172)  
- 📨 Slack: **@Hov172**  
- 🕹️ Discord: **Jay172_**  
- [LinkedIn](https://www.linkedin.com/in/jesus-a-785bb616?trk=people-guest_people_search-card)  
- 🐦 [Twitter / X (@AyalaSolutions)](https://twitter.com/AyalaSolutions)  
- <a href="https://bsky.app/profile/ayalasolutions.bsky.social"><img src="https://raw.githubusercontent.com/bluesky-social/social-app/main/assets/logo.png" width="20" alt="Bluesky Logo"></a> [@AyalaSolutions](https://bsky.app/profile/ayalasolutions.bsky.social)  
- [![Buy Me A Coffee](https://img.shields.io/badge/Buy_Me_A_Coffee-FFDD00?style=flat&logo=buy-me-a-coffee&logoColor=black)](https://buymeacoffee.com/hov172)  
- 📧 *Contact via GitHub, Social accounts issues or discussions*  

---

⭐ *If you find my tools useful, consider giving them a star to support future development!*
