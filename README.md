# Bookmark Backup Tool - v2.1 beta

**Bookmark Backup Tool** is a powerful utility for macOS designed to back up, restore, and manage your web browser bookmarks. It offers both a user-friendly graphical interface (GUI) and a comprehensive command-line interface (CLI) for power users and automation.

Whether you want to perform a quick manual backup, schedule regular automated exports, or migrate bookmarks, this tool has you covered.

** 2 Version one with the Setting tab and on without.

## Features

- **Dual Interface**: Use the intuitive GUI for easy operation or the robust CLI for scripting.
- **Multi-Browser Support**: Works with Chrome, Edge, Firefox, and Safari.
- **Profile Management**: Automatically detects and manages multiple user profiles.
- **Flexible Export/Import**:
    - Process any combination of browsers and profiles.
    - Save backups to a custom location and automatically create ZIP archives.
- **Scheduled Backups**:
    - Set up automated, recurring backup tasks (daily, weekly, or monthly).
- **Configuration Management**:
    - Export and import the app's configuration to a JSON file, making setup on a new machine easy.
- **Health Check & Utilities**:
    - Built-in tools to verify permissions, validate bookmark files, and clean up old logs.

## Supported Browsers

- Google Chrome
- Microsoft Edge
- Mozilla Firefox
- Apple Safari

## Important Note: Safari & Full Disk Access

Scheduled backups that include Safari require **Full Disk Access**. Without this permission, automated tasks will fail.

**To grant Full Disk Access:**
1.  Open **System Settings** → **Privacy & Security** → **Full Disk Access**.
2.  Click the `+` button, find **Bookmark Backup Tool**, and add it to the list.
3.  Ensure the toggle next to the app is enabled.

The application will warn you if you create a scheduled task involving Safari without the required permissions.

## Enterprise Deployment (MDM)

For enterprise environments, IT administrators can deploy the included TCC privacy profile (`config/tcc-profile.mobileconfig`) via an MDM solution. This pre-approves Full Disk Access, so end-users do not need to grant it manually. **Note:** TCC profiles can only be installed via MDM, not by a user.

## Advanced Configuration

The behavior of the Bookmark Backup Tool can be customized for managed environments by deploying a custom `Settings.plist` file. This is ideal for administrators who want to provide a simplified, pre-configured experience for users.

### Hiding the Settings Tab

To prevent users from changing settings, you can hide the "Settings" tab in the GUI.

1.  Create or edit a `Settings.plist` file.
2.  Add the following key-value pair:
\`\`\`xml
    <key>ShowSettingsTab</key>
    <false/>
\`\`\`
3.  Deploy this file to `/Library/Application Support/BookmarksBackupTool/Settings.plist` on the target machine.

When the application launches, it will read this system-level setting and hide the "Settings" tab, preventing users from modifying the default configuration.

### What's New in v2.1

- **Enhanced Settings Tab**: Added a "Choose" button next to the default backup path field, allowing users to easily select a folder location through a file picker dialog.
- **Improved Settings Management**: Fixed an issue where the `ShowSettingsTab` setting wasn't properly loaded from the bundled settings file, ensuring that administrators can reliably hide the settings tab in managed environments.

## Usage: Graphical User Interface (GUI)

Launch the application to access the main window for all primary functions.

- **Main Tab**: Perform immediate exports or imports. Select your browsers, choose a target path, and go.
- **Profiles Tab**: Lists all browser profiles automatically detected on your system.
- **Settings Tab**: Customize default behaviors, such as the default backup path, default browsers, and log retention. You can also manage and view scheduled backups from this tab.

## Usage: Command-Line Interface (CLI)

# Complete Application Usage Guide

The Bookmark Backup Tool is a comprehensive utility for managing browser bookmarks on macOS with both GUI and CLI interfaces.

## Command-Line Interface Access

The CLI can be accessed using the full path to the executable inside the application bundle.

### Direct Path Execution (Always Works)
After installing the application, you can run commands like this:
```
# Full path to the CLI command
/Applications/Bookmark\ Backup\ Tool.app/Contents/MacOS/BookmarksBackupTool --help
```

### Creating a Symlink for Convenience
For easier access, you can create a symbolic link to a directory in your system's `PATH`.
```
# Create symlink in /usr/local/bin (requires admin password)
sudo ln -s "/Applications/Bookmark Backup Tool.app/Contents/MacOS/BookmarksBackupTool" /usr/local/bin/bookmarks-backup

# Now you can use the shorter command:
bookmarks-backup --version
bookmarks-backup check
```
## CLI Command Reference

*(The following examples assume you have created the `bookmarks-backup` symlink for convenience.)*

### Basic Commands
```
# Show help information
bookmarks-backup --help

# Show application status and detected profiles
bookmarks-backup status

# List all detected browser profiles
bookmarks-backup list-profiles
```

### Exporting Bookmarks
```
# Export bookmarks from default browsers to the default location
bookmarks-backup export

# Export from specific browsers to a custom location
bookmarks-backup export --chrome --firefox --path "/Users/Shared/Backups"

# Export all profiles for all browsers and create a ZIP archive
bookmarks-backup export --all-browsers --all-profiles --zip

# Preview an export without writing files (dry run)
bookmarks-backup export --all-browsers --dry-run
```

### Importing Bookmarks
```
# Import bookmarks from a backup directory
bookmarks-backup import --all-browsers --path "/path/to/backup"

# Import from a ZIP archive, forcing the import even if a browser is running
bookmarks-backup import --all-browsers --path "/path/to/backup.zip" --force

# Skip creating a backup of existing bookmarks before import
bookmarks-backup import --all-browsers --path "/path/to/backup" --no-backup
```

### Scheduled Backups Management
```
# Create a daily scheduled backup at 10:00 AM
bookmarks-backup schedule create --all-browsers --time "10:00" --frequency Daily

# Create a weekly scheduled backup on Mondays at 9:30 AM
bookmarks-backup schedule create --all-browsers --time "09:30" --frequency Weekly --weekday 1

# Remove the existing scheduled task
bookmarks-backup schedule remove
```

### Settings Management
```
# Get a specific setting value
bookmarks-backup settings get defaultPath

# Set a new default backup path
bookmarks-backup settings set defaultPath "/Users/Shared/Backups"

# Export the entire application configuration to a JSON file
bookmarks-backup settings export --output "/path/to/config.json"

# Import an application configuration from a JSON file
bookmarks-backup settings import "/path/to/config.json"
```

### System Maintenance
```
# Apply the log retention policy and clean up old logs
bookmarks-backup cleanup-logs

# Perform a system health check to verify permissions and profiles
bookmarks-backup check
```

## Browser Support
- **Google Chrome** - Supports multiple profiles
- **Microsoft Edge** - Supports multiple profiles  
- **Mozilla Firefox** - Supports multiple profiles
- **Apple Safari** - Single profile (requires Full Disk Access for scheduled backups)

## Enterprise Deployment Features
- **MDM Deployment**: Can be deployed via Mobile Device Management solutions.
- **TCC Profile**: Pre-configured privacy profile for automatic Full Disk Access.
- **Silent Operation**: CLI mode supports `--quiet` for automated scripts.
- **JSON Output**: Structured output for integration with monitoring systems.
- **Configuration Management**: Export/import settings for consistent deployment.

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
1. Import the `tcc-profile.mobileconfig` file into your MDM solution.
2. Deploy the profile to target devices along with the Bookmark Backup Tool application.
3. Users will have Full Disk Access permissions automatically without manual intervention.

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
4. **Monitor Compliance**: Use the `check` command to verify backup configurations.

## Advanced Configuration: Hiding the Settings Tab

The behavior of the Bookmark Backup Tool can be customized for managed environments. For administrators who want to provide a simplified, pre-configured experience, the "Settings" tab in the GUI can be hidden.

1.  Create or edit a `Settings.plist` file.
2.  Add the following key-value pair:
    \`\`\`xml
    <key>ShowSettingsTab</key>
    <false/>
    \`\`\`
3.  Deploy this file to `/Library/Application Support/BookmarksBackupTool/Settings.plist` on the target machine.

When the application launches, it will read this system-level setting and hide the "Settings" tab, preventing users from modifying the default configuration.


The CLI is perfect for scripting and automation.

### Executing Commands

The executable is located inside the app bundle. After installing the app to `/Applications`, you can run commands like this:
