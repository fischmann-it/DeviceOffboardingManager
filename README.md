# ⚙️ Device Offboarding Manager

<div align="center">
  <p>
    <a href="https://twitter.com/UgurKocDe">
      <img src="https://img.shields.io/badge/Follow-@UgurKocDe-1DA1F2?style=flat&logo=x&logoColor=white" alt="Twitter Follow"/>
    </a>
    <a href="https://www.linkedin.com/in/ugurkocde/">
      <img src="https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat&logo=linkedin" alt="LinkedIn"/>
    </a>
    <img src="https://img.shields.io/github/license/ugurkocde/IntuneAssignmentChecker?style=flat" alt="License"/>
  </p>
  <a href="https://www.powershellgallery.com/packages/DeviceOffboardingManager">
      <img src="https://img.shields.io/powershellgallery/v/DeviceOffboardingManager?style=flat&label=PSGallery%20Version" alt="PowerShell Gallery Version"/>
    </a>
    <a href="https://www.powershellgallery.com/packages/DeviceOffboardingManager">
      <img src="https://img.shields.io/powershellgallery/dt/DeviceOffboardingManager?style=flat&label=PSGallery%20Downloads&color=brightgreen" alt="PowerShell Gallery Downloads"/>
    </a>
</div>

A modern PowerShell-based GUI tool for managing and offboarding devices from Microsoft Intune, Autopilot, and Entra ID (formerly Azure AD). This tool provides a streamlined interface for device lifecycle management across Microsoft services.

> **Note**: Version 0.3 is the final release of the PowerShell script. Development has moved to a native Windows app (WinUI 3) that will replace the script as version 0.4 — same features, no PowerShell setup required. Follow the progress in [Issue #60](https://github.com/ugurkocde/DeviceOffboardingManager/issues/60). The script remains available on the PowerShell Gallery and critical bugs in 0.3 will still be fixed.

## Watch the full walkthrough of the tool:

<div align="center">
      <a href="https://www.youtube.com/watch?v=CbximIIAEgc">
     <img 
      src="https://img.youtube.com/vi/CbximIIAEgc/maxresdefault.jpg" 
      alt="IntuneAssignmentChecker" 
      style="width:100%;">
      </a>
</div>

## Table of Contents

- [⚙️ Device Offboarding Manager](#️-device-offboarding-manager)
  - [Watch the full walkthrough of the tool:](#watch-the-full-walkthrough-of-the-tool)
  - [Table of Contents](#table-of-contents)
  - [🚀 Quick Start](#-quick-start)
    - [Option 1: Install from PowerShell Gallery (Recommended)](#option-1-install-from-powershell-gallery-recommended)
    - [Option 2: Manual Installation](#option-2-manual-installation)
  - [🎯 Features](#-features)
    - [🔑 Core Functionality](#-core-functionality)
    - [💻 Device Management](#-device-management)
    - [📊 Dashboard Analytics](#-dashboard-analytics)
    - [📚 Playbooks](#-playbooks)
  - [⚡ Prerequisites](#-prerequisites)
  - [🔒 Required Roles and Permissions](#-required-roles-and-permissions)
  - [🔧 Usage](#-usage)
    - [🔐 Authentication](#-authentication)
    - [🛡️ Defender for Endpoint (optional)](#️-defender-for-endpoint-optional)
    - [💻 Device Management](#-device-management-1)
    - [📊 Dashboard](#-dashboard)
    - [📚 Playbooks](#-playbooks-1)
  - [👥 Contributing](#-contributing)
  - [📄 License](#-license)

## 🚀 Quick Start

> **Important**: All commands must be run in a PowerShell 7 session. The script will not work in PowerShell 5.1 or earlier versions.

### Option 1: Install from PowerShell Gallery (Recommended)

```powershell
# Install Microsoft Graph Authentication Modul
Install-Module Microsoft.Graph.Authentication -Scope CurrentUser
```

```powershell
# Install from PowerShell Gallery
Install-Script DeviceOffboardingManager
```

```powershell
# Open a new PowerShell 7 session to run the script with
DeviceOffboardingManager
```

### Option 2: Manual Installation

```powershell
# Install Microsoft Graph Authentication Modul
Install-Module Microsoft.Graph.Authentication -Scope CurrentUser

# Download and run the script
.\DeviceOffboardingManager.ps1
```

### Update to the latest Version

```powershell
# Restart the PowerShell Session after installing the new version
Install-Script DeviceOffboardingManager -Force
```

## 🎯 Features

### 🔑 Core Functionality

- **Multi-Service Integration**: Manage devices across Intune, Autopilot, and Entra ID
- **Bulk Operations**: Support for bulk device imports and operations
- **Real-time Dashboard**: View device statistics and distribution
- **Secure Authentication**: Multiple authentication methods including interactive, certificate, and client secret

### 💻 Device Management

![Homer](media/device_offboarding.png)

- Search devices by name or serial number
- View device details including:
  - Last contact times
  - Operating system
  - Primary user
  - Management status across services
- Bulk device offboarding with confirmation
- Automatic retrieval of BitLocker/FileVault keys

### 📊 Dashboard Analytics

![Dashboard Analytics](media/dashboard.png)

- Total device counts per service
- Stale device tracking (30/90/180 days)
- Personal vs Corporate device distribution
- Platform distribution visualization
- Real-time statistics updates

### 📚 Playbooks

![Playbooks](media/playbooks.png)

- Automated device management tasks
- Pre-built reports and analyses
- Custom playbook support for specific scenarios

## ⚡ Prerequisites

1. PowerShell 7.0 or higher
2. Microsoft.Graph.Authentication module
3. Required Microsoft Graph API permissions:
   - Device.ReadWrite.All
   - DeviceManagementApps.Read.All
   - DeviceManagementConfiguration.Read.All
   - DeviceManagementManagedDevices.ReadWrite.All
   - DeviceManagementServiceConfig.ReadWrite.All
   - Group.Read.All
   - User.Read.All
   - BitlockerKey.Read.All
   - DeviceLocalCredential.Read.All (for LAPS passwords)
4. Optional: MSAL.PS module (only if you enable the Defender for Endpoint integration)

The built-in Prerequisites dialog (sidebar) checks all of this for you and can install missing modules.

## 🔒 Required Roles and Permissions

The Graph permissions above are **not sufficient on their own** when you sign in interactively — your admin account also needs directory/Intune roles, otherwise offboarding fails with `403 Forbidden`:

| Operation | Required role |
|---|---|
| Delete device from **Entra ID** | Cloud Device Administrator (or Intune Administrator) |
| Delete device from **Intune** | Intune Administrator, or an Intune RBAC role with *Managed devices – Delete* |
| Delete device from **Autopilot** / set group tags | Intune Administrator |
| Read BitLocker recovery keys | A role permitted to read BitLocker keys (e.g. Cloud Device Administrator, Intune Administrator, Helpdesk Administrator) |
| Read LAPS passwords | Cloud Device Administrator or Intune Administrator |

Since 0.3 the tool detects `403` responses and tells you which role is likely missing instead of showing raw JSON. If your tenant uses **Multi-Admin Approval (MAA)** for protected operations, deletions are reported as "Requires Multi-Admin Approval" — approve the request in Intune and re-run.

For app-only authentication (certificate or client secret), the application permissions listed under Prerequisites are sufficient; directory roles do not apply.

## 🔧 Usage

### 🔐 Authentication

The tool supports four authentication methods:

1. **Interactive Login**: Best for admin users with appropriate permissions
2. **Device Code Login**: Interactive login without a browser redirect — use this if the normal interactive login fails with localhost-redirect or WAM errors (common on locked-down machines and in remote sessions)
3. **Certificate-based**: For automated or service principal authentication
4. **Client Secret**: Alternative service principal authentication method

To connect:

1. Click "Connect to MS Graph" in the sidebar
2. Choose your authentication method
3. Provide required credentials
4. Verify connection status in the tenant information section

Certificate and client secret configurations can be saved and are auto-loaded on the next start (stored in `%LocalAppData%/DeviceOffboardingManager`; the client secret itself is never persisted).

### 🛡️ Defender for Endpoint (optional)

Offboarding devices from Microsoft Defender for Endpoint is available as an opt-in integration, disabled by default:

1. Open the **Prerequisites** dialog from the sidebar
2. Enable the **Defender for Endpoint integration** toggle (persisted in `settings.json`)
3. Install the optional **MSAL.PS** module when prompted

Once enabled, Defender appears as an additional offboarding target. It requires `WindowsDefenderATP` permissions (`Machine.ReadWrite.All`, `Machine.Offboard`) on your app registration. Both app-only (certificate / client secret) and delegated authentication are supported.

### 💻 Device Management

1. **Search for Devices**:

   - Select search type (Device name / Serial number / Device ID / Contains partial match)
   - Enter search terms (supports multiple values with comma separation)
   - Click Search to retrieve device information
   - Filter the results grid live via the filter boxes above each column; shift-click checkboxes to select ranges

2. **Bulk Import**:

   - Click "Bulk Import"
   - Select a CSV/TXT file containing device names or serial numbers
   - Verify imported devices in the search field

3. **Device Offboarding**:

   - Select devices in the results grid
   - Click "Offboard device(s)"
   - Review the confirmation dialog (shows the exact Entra/Intune/Autopilot IDs that will be affected, plus co-management warnings)
   - Note any encryption recovery keys and LAPS passwords
   - Confirm the operation

4. **Set Autopilot Group Tags**:
   - Select devices in the results grid
   - Click "Set Group Tag" to assign or clear the Autopilot group tag for all selected devices

### 📊 Dashboard

The dashboard provides real-time insights into your device management environment:

- Device counts across services
- Stale device tracking
- Ownership distribution
- Platform distribution
- Quick access to common tasks

### 📚 Playbooks

Automated tasks for common scenarios:

- Find Autopilot devices not in Intune
- List Intune devices not in Autopilot
- Generate corporate device inventory
- View personal device inventory
- Analyze stale devices
- OS-specific, outdated-OS, and end-of-life-OS device reports
- BitLocker and FileVault key reports
- Corporate identifier stale report

Playbooks are bundled inside the script, so they also work when installed via `Install-Script` from the PowerShell Gallery.

## 👥 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
