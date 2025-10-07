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

The CLI is perfect for scripting and automation.

### Executing Commands

The executable is located inside the app bundle. After installing the app to `/Applications`, you can run commands like this:
