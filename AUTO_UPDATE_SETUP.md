# ECCAT Desktop Apps - Auto-Update System Setup Guide

## Overview

This document explains how to set up and use the automatic update system for ECCAT desktop applications using `desktop_updater` package and GitHub Releases.

## Architecture

- **Package**: `desktop_updater` (v1.4.0)
- **Hosting**: GitHub Releases (https://github.com/waytoo-average/desktop_apps_updates)
- **Local Path**: `D:\ECCAT_PERSONAL_PROJECTS\app_project\app\desktop_apps_updates`
- **Update Detection**: `app-archive.json` hosted on GitHub Releases
- **Delta Updates**: Only changed files are downloaded (Windows/Linux)

## Components

### 1. Desktop Apps with Update Support

All four desktop apps now include:
- **desktop_updater** dependency
- **SettingsScreen** with update widget
- **Settings button** in AppBar

Apps:
- eccat_admin (v1.0.0)
- eccat_doctor (v1.0.1)
- eccat_student (v1.0.0)
- eccat_labs (v0.1.0)

### 2. Release Automation Tool

**Location**: `suggestions_manager/lib/screens/release_automation_screen.dart`

Features:
- Detects version changes in pubspec.yaml files
- Builds Windows releases using `desktop_updater`
- Shows "Update Available" badge when version changes
- "Build All" button for batch releases
- Real-time build log output

### 3. app-archive.json

**Location**: `app/app-archive.json`

Structure:
```json
{
  "appName": "ECCAT Desktop Apps",
  "description": "ECCAT System Desktop Applications - Auto-update archive",
  "items": [
    {
      "version": "1.0.0",
      "shortVersion": 1,
      "changes": [
        {
          "type": "feat",
          "message": "Initial release with auto-update support"
        }
      ],
      "date": "2026-06-09",
      "mandatory": false,
      "url": "https://github.com/yourusername/your-repo/releases/download/v1.0.0/eccat_admin-1.0.0+1-windows",
      "platform": "windows"
    }
  ]
}
```

## Setup Instructions

### Step 1: Install desktop_updater CLI

```bash
dart pub global activate desktop_updater
```

### Step 2: Configure GitHub Repository

**Repository**: https://github.com/waytoo-average/desktop_apps_updates

The `appArchiveUrl` is already configured in all apps:
```dart
appArchiveUrl: Uri.parse(
  'https://github.com/waytoo-average/desktop_apps_updates/releases/download/latest/app-archive.json',
),
```

### Step 3: Build First Release

For each app:

```bash
cd eccat_admin
dart run desktop_updater:release windows
```

This creates a `dist/1/1.0.0+1-windows` folder containing:
- `hashes.json` - File hashes for delta updates
- All app files

### Step 4: Upload to GitHub Releases

1. Create a new release on GitHub: `v1.0.0`
2. Upload the entire `dist/1/1.0.0+1-windows` folder
3. Upload `app-archive.json` to the release
4. Note the download URL format: `https://github.com/waytoo-average/desktop_apps_updates/releases/download/v1.0.0/`

### Step 5: Update app-archive.json

For each app release, add an entry to `app-archive.json`:

```json
{
  "version": "1.0.0",
  "shortVersion": 1,
  "changes": [
    {
      "type": "feat",
      "message": "Your change description"
    }
  ],
  "date": "2026-06-09",
  "mandatory": false,
  "url": "https://github.com/waytoo-average/desktop_apps_updates/releases/download/v1.0.0/eccat_admin-1.0.0+1-windows",
  "platform": "windows"
}
```

## Release Workflow

### Using Release Automation Tool (Recommended)

1. **Create GitHub Personal Access Token**:
   - Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
   - Generate new token with `repo` scope
   - Copy the token (starts with `ghp_`)

2. **Open Release Automation Tool**:
   - Open **Suggestions Manager** app
   - Click the **Build** icon (hammer) in AppBar
   - Navigate to **Release Automation** screen

3. **Enter GitHub Token**:
   - Paste your GitHub token in the input field
   - The token is required for creating releases and uploading assets

4. **Update Versions (Optional)**:
   - Click the **edit icon** next to any app's version number
   - Enter new version in format `VERSION+BUILD_NUMBER` (e.g., `1.0.0+1`)
   - Click **Save** to update the pubspec.yaml file
   - The version is immediately reflected in the UI

5. **Build and Upload**:
   - The tool automatically:
     - Loads current versions from all pubspec.yaml files
     - Shows "Update Available" badge if version changed
     - Builds Windows releases using `desktop_updater`
     - Creates GitHub release automatically
     - Uploads dist folder contents to GitHub
     - Uploads app-archive.json to the release
   - Click **Build** for individual apps or **Build All Releases** for batch processing

6. **Monitor Progress**:
   - Watch the log output for real-time progress
   - Check for any errors in the log

### Manual Release Process (If Automation Fails)

1. **Update version** in `pubspec.yaml`:
   ```yaml
   version: 1.0.1+2  # version+build_number
   ```

2. **Build release**:
   ```bash
   cd eccat_admin
   dart run desktop_updater:release windows
   ```

3. **Upload to GitHub**:
   - Create release `v1.0.1` on GitHub
   - Upload `dist/1/1.0.1+2-windows` folder
   - Upload updated `app-archive.json`

4. **Test update**:
   - Run previous version of app
   - Go to Settings
   - Update widget should show new version
   - Click to update

## Version Numbering

Format: `VERSION+BUILD_NUMBER`

- **VERSION**: Semantic version (e.g., 1.0.0, 1.0.1)
- **BUILD_NUMBER**: Incrementing integer (e.g., 1, 2, 3)

Example:
```yaml
version: 1.0.0+1  # First release
version: 1.0.1+2  # Bug fix release
version: 1.1.0+3  # Feature release
```

## Change Types

Use these types in `app-archive.json`:
- `feat` - New feature
- `fix` - Bug fix
- `chore` - Maintenance task
- `docs` - Documentation
- `style` - Code style changes
- `refactor` - Code refactoring
- `perf` - Performance improvement
- `test` - Test additions
- `build` - Build system changes
- `ci` - CI configuration

## Hosting Options

### GitHub Releases (Recommended)

**Pros**:
- Free hosting
- Built-in version management
- Easy to use
- Good for public projects

**Cons**:
- 2GB file size limit per release
- Manual upload required

### GitHub Pages

**Pros**:
- Free hosting
- Can serve JSON files
- Easy to automate

**Cons**:
- No built-in version management
- Need to manage file structure manually

### S3/Cloud Storage

**Pros**:
- High reliability
- Can automate uploads
- No file size limits

**Cons**:
- Cost (though minimal for small projects)
- Requires AWS account

## Troubleshooting

### Update Not Showing

1. Check `appArchiveUrl` is correct in SettingsScreen
2. Verify `app-archive.json` is accessible via browser
3. Check version numbers in `app-archive.json` vs app
4. Ensure app has internet connection

### Build Fails

1. Ensure `desktop_updater` CLI is installed: `dart pub global activate desktop_updater`
2. Check Flutter SDK is in PATH
3. Verify app builds normally: `flutter build windows`
4. Check build log in Release Automation screen

### Download Fails

1. Verify GitHub release is public
2. Check URL format in `app-archive.json`
3. Ensure all files in `dist/` folder were uploaded
4. Check `hashes.json` exists and is valid

## Security Considerations

- **No signing required** for Windows (desktop_updater uses SHA-256 hashes)
- **Public releases** are fine for ECCAT (public educational system)
- **Private releases** would require additional authentication

## Future Enhancements

- [ ] Automate GitHub Release uploads via GitHub API
- [ ] Add macOS support (requires code signing)
- [ ] Add Linux support
- [ ] Implement beta channel testing
- [ ] Add rollback capability
- [ ] Schedule automatic update checks

## Support

For issues or questions:
1. Check desktop_updater documentation: https://pub.dev/packages/desktop_updater
2. Review GitHub issues: https://github.com/MarlonJD/flutter_desktop_updater
3. Contact system administrator
