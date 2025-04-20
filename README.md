# 🔧 ENVOY - Lightweight User Environment Manager

This tool automates the deployment or execution of configurations during logon on Windows machines. It is mainly built for Intune Managed devices where certain actions are not natively possible from Intune. During execution it connects to Microsoft Graph using an App Registration and grabs the group memberships from the user. The script performs tasks such as drive mapping, printer mapping, registry key management, File Actions (copy/move/delete/rename) and starts executables based on Entra ID group membership.

&nbsp;

> [!NOTE]
> Envoy is still in development. More information available soon!
&nbsp;

# 📺 Demo

Watch the video on Youtube

[![Watch the video on Youtube](https://i.ytimg.com/vi/BE4vtI_WJGE/hq720.jpg?sqp=-oaymwEnCNAFEJQDSFryq4qpAxkIARUAAIhCGAHYAQHiAQoIGBACGAY4AUAB&rs=AOn4CLC1hCvaNLwTVhb9RM0J46CS5sLsYw)](https://youtu.be/BE4vtI_WJGE)

&nbsp;

# 📑 Contents
- [🔧 ENVOY - Lightweight User Environment Manager](#-envoy---lightweight-user-environment-manager)
- [📺 Demo](#-demo)
- [📑 Contents](#-contents)
- [🚀 Core Components](#-core-components)
- [🔜 Roadmap](#-roadmap)
- [💥 Functions](#-functions)
    - [📰 Write-Log](#-write-log)
    - [📁 Deploy-DriveMappings](#-deploy-drivemappings)
    - [📘 Deploy-RegistryKeys](#-deploy-registrykeys)
    - [⏳ Deploy-Executables](#-deploy-executables)
    - [💾 Deploy-FileActions](#-deploy-fileactions)
    - [📠 Deploy-Printers](#-deploy-printers)
  - [✅ Execution](#-execution)
  - [✅ Delayed Execution](#-delayed-execution)
  - [🔃 Envoy Refresh](#-envoy-refresh)
  - [⚠️ Dependencies](#️-dependencies)
  - [💻 Installation](#-installation)
  - [⏩ Contributing / Feature Request](#-contributing--feature-request)

&nbsp;

# 🚀 Core Components

- **Microsoft Graph Authentication**: The script uses the `Microsoft.Graph` modules to interact with Microsoft Graph APIs.
- **Configuration File**: Reads settings from `Config.json` located at `C:\ProgramData\Envoy\Config.json`.
- **Authentication**: Connects to Microsoft Graph using `TenantId`, `AppId`, and `AppSecret` from the configuration file.

&nbsp;

# 🔜 Roadmap
- TBA
  
&nbsp;

# 💥 Functions

### 📰 Write-Log

- **Purpose:** Logs messages to a user-specific log file located at `C:\ProgramData\Envoy\Logging\<username>\User.log`.

- **Key Features:** Ensures the log directory exists and appends timestamped log entries.

&nbsp;

### 📁 Deploy-DriveMappings

**Purpose**: Maps or removes network drives based on configuration.

**Key Features:**
  - Reads drive mapping configurations from `Config.json`.
  - Checks user group memberships to determine eligibility.
  - Supports adding and removing drive mappings.
  - Logs success or failure of each operation.
  - Priority handling for situations with conflicting actions (e.g. same drive letters).

&nbsp;

**Example:**
```
"Drives": [
      {
        "DriveLetter": "X",
        "UNCPath": "\\\\storageaccount.file.core.windows.net\\share1",
        "Group": "GG - Share 1",
        "Description": "Test Drive",
        "Priority": 1,
        "Action": "add"
      },
      {
        "DriveLetter": "Y",
        "UNCPath": "\\\\fileserver\\share2",
        "Group": "GG - Share 2",
        "Description": "Test Drive 2",
        "Priority": 1,
        "Action": "add"
      }
    ],
```

**Usage:**
| Setting           | Values      | Description  | 
|------------------|-----|-----------------------|
| DriveLetter      | x,y,z, etc.  | Configure the desired drive mapping letter.                   |
| UNCPath           |\\\\server.domain.local\\share  | Configure the desired UNC path. Don't forget double slashes for JSON |
| Group | e.g. "GG - Sales Team" | Configure the desired Entra ID group. Users in this group will receive this drive mapping. Leave the group empty for "everyone". |
| Description | Text | Fill in a description. E.g. Sales Drive, Markering Team. |
| Priority | 1,2,3,4,5,6, etc | Conflicts with drive mappings can occur if a user is a member of multiple groups with the same drive letter as result. |Prio 1 is the lowest, higher winns. |
| Action | Add or Remove | Define the action for the drive mapping |

&nbsp;

### 📘 Deploy-RegistryKeys

**Purpose:**  Adds or removes registry keys based on configuration.

**Key Features:**
  - Reads registry configurations from `Config.json`.
  - Checks user group memberships to determine eligibility.
  - Ensures registry keys exist before setting values.
  - Supports adding and removing registry keys.
  - Logs success or failure of each operation.

&nbsp;

**Example:**
```
    "Registries": [
      {
        "Key": "HKCU:\\Software\\TestKey",
        "ValueName": "TestValue1",
        "ValueType": "DWORD",
        "ValueData": 1,
        "Group": "GG - Registry - TestKey",
        "Action": "add"
      },
      {
        "Key": "HKCU:\\Software\\TestKey",
        "ValueName": "TestValue2",
        "ValueType": "STRING",
        "ValueData": "2",
        "Group": "",
        "Action": "add"
      }
    ],
```

**Usage:**
| Setting           | Values      | Description  | 
|------------------|-----|-----------------------|
| Key | HKCU:\\Path\\ | Fill in the desired HKCU Path |
| ValueName | Text | Fill in the desired value name |
| ValueType | DWORD, STRING, etc | Define the desired type |
| ValueData | Data | Define the desired data. Decimal, text, path's, etc. |
| Group | e.g. "GG - Sales Team" | Configure the desired Entra ID group. Users in this group will receive this registry setting. Leave the group empty for "everyone". |
| Action | Add or Remove | Define the action for the registry key |

&nbsp;

### ⏳ Deploy-Executables

**Purpose:** Starts executables with optional arguments based on configuration.

**Key Features:**
  - Reads executable configurations from `Config.json`.
  - Checks user group memberships to determine eligibility.
  - Supports starting executables with or without arguments.
  - Logs success or failure of each operation.

&nbsp;
**Example:**
```
    "Executables": [
      {
        "FilePath": "C:\\ProgramFiles\\CustomApp\\Setup.msi",
        "Arguments": "/qn",
        "Group": "GG - Custom App"
      },
      {
        "FilePath": "C:\\Windows\\System32\\calc.exe",
        "Arguments": "",
        "Group": "GG - Calculator"
      }     
    ],
```

&nbsp;

**Usage:**
| Setting           | Values      | Description  | 
|------------------|-----|-----------------------|
| FilePath | C:\Folder\File.exe | Configure the file path |
| Arguments | /qn, /silent, etc | Supports arguments belonging to the application |
| Group | e.g. "GG - Sales Team" | Configure the desired Entra ID group. Users in this group will automatically launch the configured application. Leave the group empty for "everyone". |

&nbsp;

### 💾 Deploy-FileActions

**Purpose:** Performs file operations (copy, delete, rename, move) based on configuration.

**Key Features:**
  - Reads file action configurations from `Config.json`.
  - Checks user group memberships to determine eligibility.
  - Supports the following actions:
    - Copy: Copies files to a destination.
    - Delete: Deletes files.
    - Rename: Renames files.
    - Move: Moves files to a new location. 
  - Logs success or failure of each operation.

&nbsp;

**Example:**
```
    "FileActions": [
      {
        "FileActionType": "copy",
        "SourcePath": "C:\\Temp\\Source\\File.txt",
        "DestinationPath": "C:\\Temp\\Destination\\File.txt",
        "NewName": "NewFileName.txt",
        "Group": "GG - FileAction - Copy"
      },
      {
        "FileActionType": "rename",
        "SourcePath": "C:\\Temp\\Source\\File.txt",
        "DestinationPath": "C:\\Temp\\Destination\\File.txt",
        "NewName": "NewFileName.txt",
        "Group": "GG - FileAction - Rename"
      },
      {
        "FileActionType": "move",
        "SourcePath": "C:\\Temp\\Source\\File.txt",
        "DestinationPath": "C:\\Temp\\Destination\\File.txt",
        "NewName": "NewFileName.txt",
        "Group": "GG - FileAction - Move"
      },
      {
        "FileActionType": "delete",
        "SourcePath": "C:\\Temp\\Source\\File.txt",
        "DestinationPath": "C:\\Temp\\Destination\\File.txt",
        "NewName": "NewFileName.txt",
        "Group": "GG - FileAction - Delete"
      }                  
    ],
```

**Usage:**
| Setting           | Values      | Description  | 
|------------------|-----|-----------------------|
| FileActionType | copy, rename, move, delete | Configure the file action |
| SourcePath | C:\Folder\Source\File.ini | Being used for actions: copy, rename, move, delete |
| DestinationPath | C:\Folder\Destination\File.ini | Being used for actions: copy, move |
| NewName | FileName | Set the desired new file name. Being used for actions: rename |
| Group | e.g. "GG - Sales Team" | Configure the desired Entra ID group. Users in this group will automatically execute the file actions. Leave the group empty for "everyone". |

&nbsp;

### 📠 Deploy-Printers

**Purpose:** Adds or removes printers based on configuration.

**Key Features:**
  - Reads printer configurations from `Config.json`.
  - Checks user group memberships to determine eligibility.
  - Supports adding and removing printers.
  - Logs success or failure of each operation.

&nbsp;
**Example:**
```
    "Printers": [
      {
        "PrinterPath": "\\\\PRINTSRV.domain.local\\FollowMe",
        "Group": "GG - Printers - FollowMe",
        "Action": "add"
      },
      {
        "PrinterPath": "\\\\PRINTSRV.domain.local\\PRT01",
        "Group": "",
        "Action": "remove"
      }
    ] 
```
**Usage:**
| Setting           | Values      | Description  | 
|------------------|-----|-----------------------|
| PrinterPath | \\\\server\\printer | Configure the UNC path for the desired printer |
| Group | e.g. "GG - Sales Team" | Configure the desired Entra ID group. Users in this group will automatically add or remove the printer queue. Leave the group empty for "everyone". |
| Action | Add or Remove | Define if the printer queue should be added or removed |


&nbsp;

## ✅ Execution

The script executes the following tasks sequentially while logging on. The scheduled task created by `Envoy-core.ps1` will execute these functions.

1. **Drive Mapping**: Execute `Deploy-DriveMappings` to manage network drives.
2. **Registry Key Management**: Execute `Deploy-RegistryKeys` to manage registry keys.
3. **Process execution**: Execute `Deploy-ProcessExecution` to manage process executions.
4. **File Actions**: Execute `Deploy-FileActions` to manage file actions.
5. **Printer mapping**: Execute `Deploy-PrinterMappings` to manage printer mappings.
6. **All functions**: Execute `Invoke-UEMDeployment` to manage all functions at once.

&nbsp;

## ✅ Delayed Execution


If you need to implement a delayed execution in your environment, you can customize the code accordingly. Locate `$delaySeconds` in the `Envoy-logon.ps1` file. Set the desired delay and remove any hashtags associated with the delay (which is disabled by default).

&nbsp;

## 🔃 Envoy Refresh

Configurations can be refreshed during an active user session. A shortcut in the public start menu allows you to re-launch Envoy. All configurations will be reassessed and executed by referencing the Config.JSON file.

![Refresh](Common/EnvoyRefresh.png)

## ⚠️ Dependencies

Requires the following PowerShell modules. Installation will be handled by `Envoy-core.ps1`.

- `Microsoft.Graph.Authentication`
- `Microsoft.Graph.Groups`
- `Microsoft.Graph.Users`

&nbsp;

> [!NOTE]
> Installation of these modules is being handled by Envoy-core.ps1 or either an installation method (e.g. MSI). This has yet to be defined.

&nbsp;

## 💻 Installation
To be defined

&nbsp;

## ⏩ Contributing / Feature Request
Contributions are welcome! Please open an issue or reach out to me directly to discuss.
