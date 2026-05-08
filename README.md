# Setting Up Binaries Repository Publishing

This guide explains how to configure GitHub Actions to publish portable binaries to the public `cryptween/cryptween-binaries` repository.

## Overview

The workflow automatically publishes portable packages to the `cryptween/cryptween-binaries` repository as GitHub Releases whenever you push a version tag (e.g., `v0.1.10`).

**Published binaries:**
- `Cryptween-linux-unpacked-{version}.tar.gz` - Linux portable package
- `Cryptween-win-unpacked-{version}.zip` - Windows portable package  
- `Cryptween-mac-unpacked-{version}.zip` - macOS portable package

---

## Download and Install

Cryptween is distributed as portable binaries - no installation required! Just download, extract, and run.

### Windows

1. **Download**:
   - Go to [Releases](https://github.com/cryptween/cryptween-binaries/releases)
   - Download the latest `Cryptween-win-unpacked-{version}.zip`

2. **Extract**:
   - Right-click the downloaded ZIP file
   - Select "Extract All..."
   - Choose a destination folder (e.g., `C:\Program Files\Cryptween` or `C:\Users\YourName\Cryptween`)
   - Click "Extract"

3. **Run**:
   - Open the extracted folder
   - Double-click `Cryptween.exe` to launch the application

**Optional**: Create a desktop shortcut by right-clicking `Cryptween.exe` → Send to → Desktop (create shortcut)

### Linux

1. **Download**:
   ```bash
   # Visit https://github.com/cryptween/cryptween-binaries/releases
   # Or download directly (replace VERSION with the latest version number):
   wget https://github.com/cryptween/cryptween-binaries/releases/download/vVERSION/Cryptween-linux-unpacked-VERSION.tar.gz
   ```

2. **Extract**:
   ```bash
   # Extract to your preferred location
   tar -xzf Cryptween-linux-unpacked-VERSION.tar.gz -C ~/Applications/
   # Or to /opt if you have permissions:
   # sudo tar -xzf Cryptween-linux-unpacked-VERSION.tar.gz -C /opt/
   ```

3. **Run**:
   ```bash
   # Navigate to the extracted folder
   cd ~/Applications/Cryptween
   
   # Make the executable runnable (if needed)
   chmod +x cryptween
   
   # Launch the application
   ./cryptween
   ```

**Optional**: Add to your application menu or create a desktop entry by creating `~/.local/share/applications/cryptween.desktop`:
```ini
[Desktop Entry]
Name=Cryptween
Comment=A network built from shared trading positions
Exec=/home/YOUR_USERNAME/Applications/Cryptween/cryptween
Icon=/home/YOUR_USERNAME/Applications/Cryptween/resources/app/icon.png
Terminal=false
Type=Application
Categories=Network;Finance;
```

### macOS

1. **Download**:
   - Go to [Releases](https://github.com/cryptween/cryptween-binaries/releases)
   - Download the latest `Cryptween-mac-unpacked-{version}.zip`

2. **Extract**:
   ```bash
   # Extract using Terminal
   unzip Cryptween-mac-unpacked-VERSION.zip -d ~/Applications/
   
   # Or double-click the ZIP file in Finder
   # Then drag the Cryptween.app to your Applications folder
   ```

3. **Run**:
   - Open **Applications** folder
   - Double-click **Cryptween.app**
   - If you see a security warning, go to **System Preferences** → **Security & Privacy** → Click "Open Anyway"

**Note**: On first run, macOS may show a security prompt because the app is not signed. This is normal for portable applications. Click "Open" to proceed.

### Updating to a New Version

To update Cryptween:
1. Download the latest version from [Releases](https://github.com/cryptween/cryptween-binaries/releases)
2. Extract to a new folder or replace the old folder
3. Your user data and settings are stored separately and will be preserved

### Troubleshooting (User)

#### macOS: "App is damaged" or "Cannot be opened" errors

macOS has several security features that block unsigned portable applications. Here are solutions for common issues:

**Issue 1: "Cryptween is damaged and can't be opened. You should move it to the Trash"**

This is caused by the quarantine attribute macOS applies to downloaded files.

**Solution**:
```bash
# Remove the quarantine attribute
xattr -cr /path/to/Cryptween.app

# Example:
xattr -cr ~/Applications/Cryptween.app
```

Then try opening the app again.

**Issue 2: "Cannot be opened because the developer cannot be verified"**

This is Gatekeeper blocking the unsigned app.

**Solution**:
1. Try to open the app (it will be blocked)
2. Go to **System Preferences/Settings** → **Security & Privacy** → **General** tab
3. You'll see a message about Cryptween being blocked
4. Click **"Open Anyway"**
5. Confirm by clicking **"Open"** in the dialog

**Alternative solution** (disable Gatekeeper temporarily):
```bash
# Disable Gatekeeper
sudo spctl --master-disable

# Open Cryptween.app normally
# After opening successfully once, re-enable Gatekeeper:
sudo spctl --master-enable
```

**Issue 3: "Cryptween can't be opened because Apple cannot check it for malicious software"**

This is a notarization check on newer macOS versions (10.15 Catalina and later).

**Solution**: Same as Issue 1 - remove quarantine attribute:
```bash
xattr -cr ~/Applications/Cryptween.app
```

**Issue 4: App opens but shows permission errors or doesn't work correctly**

This can happen due to app translocation (macOS moves apps to random locations).

**Solution**:
```bash
# Remove quarantine and extended attributes
xattr -cr ~/Applications/Cryptween.app

# If the app is in Downloads, move it to Applications folder first
mv ~/Downloads/Cryptween.app ~/Applications/
xattr -cr ~/Applications/Cryptween.app
```

**Note**: These security warnings are normal for unsigned portable applications. Once you use any of the above solutions, macOS will remember your choice and won't block the app again.

#### Windows: "Windows protected your PC" warning

**Solution**:
1. Click **"More info"**
2. Click **"Run anyway"**

This is normal for unsigned portable applications.

---
