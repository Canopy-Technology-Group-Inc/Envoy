# 🔧 ENVOY - Lightweight User Environment Manager

This tool automates the deployment or execution of configurations during logon on Windows machines. It is mainly built for Intune Managed devices where certain actions are not natively possible from Intune. During execution it connects to Microsoft Graph using an App Registration and grabs the group memberships from the user. The script performs tasks such as drive mapping, printer mapping, registry key management, File Actions (copy/move/delete/rename) and starts executables based on Entra ID group membership.

&nbsp;

> [!NOTE]
> The original idea for this tool was inspired by a former colleague. After leaving that company, I found myself frequently missing some kind of UEM tool, so I decided to recreate a simplified version on my own.

&nbsp;

# 📑 Contents
- [🔧 ENVOY - Lightweight User Environment Manager](#-envoy---lightweight-user-environment-manager)
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
  - [⚠️ Dependencies](#️-dependencies)
  - [💻 Installation](#-installation)
  - [⏩ Contributing](#-contributing)

&nbsp;

# 🚀 Core Components

- **Microsoft Graph Authentication**: The script uses the `Microsoft.Graph` modules to interact with Microsoft Graph APIs.
- **Configuration File**: Reads settings from `Config.xml` located at `C:\ProgramData\Envoy\Config.xml`.
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
  - Reads drive mapping configurations from `Config.xml`.
  - Checks user group memberships to determine eligibility.
  - Supports adding and removing drive mappings.
  - Logs success or failure of each operation.
  - Priority handling for situations with conflicting actions (e.g. same drive letters).

&nbsp;
**Example:**
```
    <Drives>
        <Drive>
            <DriveLetter>X</DriveLetter> <!-- Drive letter to assign -->
            <UNCPath>\\server.file.core.windows.net\marketing</UNCPath> <!-- UNC path to the share -->
            <Group>GG - File Share Marketing</Group> <!-- Group that should have access to the drive. Leave empty for everyone -->
            <Description>Marketing Drive</Description> <!-- Description for the drive -->
            <Priority>1</Priority> <!-- Priority handling for conflicts. 1 is lowest. Higher wins -->
            <Action>add</Action> <!-- add or remove -->
        </Drive>
        <Drive>
            <DriveLetter>Y</DriveLetter> <!-- Drive letter to assign -->
            <UNCPath>\\server.file.core.windows.net\sales</UNCPath> <!-- UNC path to the share -->
            <Group>GG - File Share Sales</Group> <!-- Group that should have access to the drive. Leave empty for everyone -->
            <Description>Sales Drive</Description> <!-- Description for the drive -->
            <Priority>2</Priority> <!-- Priority handling for conflicts. 1 is lowest. Higher wins -->
            <Action>add</Action> <!-- add or remove -->
        </Drive>       
    </Drives>
```
&nbsp;

### 📘 Deploy-RegistryKeys

**Purpose:**  Adds or removes registry keys based on configuration.

**Key Features:**
  - Reads registry configurations from `Config.xml`.
  - Checks user group memberships to determine eligibility.
  - Ensures registry keys exist before setting values.
  - Supports adding and removing registry keys.
  - Logs success or failure of each operation.

&nbsp;
**Example:**
```
    <Registries>
        <Registry>
            <Key>HKCU:\Software\TestKey1</Key> <!-- Registry key to create. Use the full Path. -->
            <ValueName>TestValue1</ValueName> <!-- Name of the registry setting to create -->
            <ValueType>DWORD</ValueType> <!-- Type of the value (DWORD, STRING, etc.) -->
            <ValueData>1</ValueData> <!-- Data for the value -->
            <Group>GG - Registry Key Test Add</Group> <!-- Group that should have the registry key. Leave empty for everyone -->
            <Action>add</Action> <!-- add or remove. Add could be used for modify actions -->
        </Registry>
        <Registry>
            <Key>HKCU:\Software\TestKey2</Key> <!-- Registry key to create. Use the full Path. -->
            <ValueName>TestValue2</ValueName> <!-- Name of the registry setting to create -->
            <ValueType>DWORD</ValueType> <!-- Type of the value (DWORD, STRING, etc.) -->
            <ValueData>2</ValueData> <!-- Data for the value -->
            <Group>GG - Registry Key Test Remove</Group> <!-- Group that should have the registry key. Leave empty for everyone -->
            <Action>remove</Action> <!-- add or remove. Add could be used for modify actions -->
        </Registry>
    </Registries>
```
&nbsp;

### ⏳ Deploy-Executables

**Purpose:** Starts executables with optional arguments based on configuration.

**Key Features:**
  - Reads executable configurations from `Config.xml`.
  - Checks user group memberships to determine eligibility.
  - Supports starting executables with or without arguments.
  - Logs success or failure of each operation.

&nbsp;
**Example:**
```
    <Executables>
        <Executable>
            <FilePath>C:\Windows\System32\notepad.exe</FilePath> <!-- Path to the executable -->
            <Arguments></Arguments> <!-- Arguments to pass to the executable -->
            <Group>GG - Executable Notepad</Group> <!-- Group that should launch the executable. Leave empty for everyone -->
        </Executable>
        <Executable>
            <FilePath>C:\ProgramFiles\CustomApp\Setup.msi</FilePath> <!-- Path to the executable -->
            <Arguments>/quiet /norestart</Arguments> <!-- Arguments to pass to the executable -->
            <Group>GG - CustomApp</Group> <!-- Group that should launch the executable. Leave empty for everyone -->
        </Executable>
    </Executables>
```
&nbsp;

### 💾 Deploy-FileActions

**Purpose:** Performs file operations (copy, delete, rename, move) based on configuration.

**Key Features:**
  - Reads file action configurations from `Config.xml`.
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
    <FileActions>
        <FileAction>
            <FileActionType>move</FileActionType> <!-- Type of file action (copy, delete, rename, move) -->
            <SourcePath>C:\Temp\Source\File.txt</SourcePath> <!-- Source path for the file action (copy/move/delete action) -->
            <DestinationPath>C:\Temp\Destination\File.txt</DestinationPath> <!-- Destination path for the file action (copy/move action) -->
            <NewName>NewFileName.txt</NewName> <!-- New name for the file (rename action). Being ignored for other Actions -->
            <Group>FileAction - EID - Test</Group> <!-- Group that should have the file action. Leave empty for everyone -->
        </FileAction>
        <FileAction>
            <FileActionType>copy</FileActionType> <!-- Type of file action (copy, delete, rename, move) -->
            <SourcePath>C:\Temp\Source\File.txt</SourcePath> <!-- Source path for the file action (copy/move/delete action) -->
            <DestinationPath>C:\Temp\Destination\File.txt</DestinationPath> <!-- Destination path for the file action (copy/move action) -->
            <NewName>NewFileName.txt</NewName> <!-- New name for the file (rename action). Being ignored for other Actions -->
            <Group>GG - Copy Test File</Group> <!-- Group that should have the file action. Leave empty for everyone -->
        </FileAction>  
        <FileAction>
            <FileActionType>rename</FileActionType> <!-- Type of file action (copy, delete, rename, move) -->
            <SourcePath>C:\Temp\Source\File.txt</SourcePath> <!-- Source path for the file action (copy/move/delete action) -->
            <DestinationPath>C:\Temp\Destination\File.txt</DestinationPath> <!-- Destination path for the file action (copy/move action) -->
            <NewName>NewFileName.txt</NewName> <!-- New name for the file (rename action). Being ignored for other Actions -->
            <Group>GG - Copy Test File</Group> <!-- Group that should have the file action. Leave empty for everyone -->
        </FileAction>
        <FileAction>
            <FileActionType>delete</FileActionType> <!-- Type of file action (copy, delete, rename, move) -->
            <SourcePath>C:\Temp\Source\File.txt</SourcePath> <!-- Source path for the file action (copy/move/delete action) -->
            <DestinationPath>C:\Temp\Destination\File.txt</DestinationPath> <!-- Destination path for the file action (copy/move action) -->
            <NewName>NewFileName.txt</NewName> <!-- New name for the file (rename action). Being ignored for other Actions -->
            <Group>GG - Copy Test File</Group> <!-- Group that should have the file action. Leave empty for everyone -->
        </FileAction>                         
    </FileActions>
```
&nbsp;

### 📠 Deploy-Printers

**Purpose:** Adds or removes printers based on configuration.

**Key Features:**
  - Reads printer configurations from `Config.xml`.
  - Checks user group memberships to determine eligibility.
  - Supports adding and removing printers.
  - Logs success or failure of each operation.

&nbsp;
**Example:**
```
    <Printers>  
        <Printer>
            <PrinterPath>\\DC01\PRTTST01</PrinterPath> <!-- Path to the printer (UNC) -->
            <Group>GG - Printer PRTTST01</Group> <!-- Group that should have the printer. Leave empty for everyone -->
            <Action>add</Action> <!-- add or remove -->
        </Printer>
        <Printer>
            <PrinterPath>\\DC01\PRTTST02</PrinterPath> <!-- Path to the printer (UNC) -->
            <Group></Group> <!-- Group that should have the printer. Leave empty for everyone -->
            <Action>remove</Action> <!-- add or remove -->
        </Printer>        
    </Printers>  
```
&nbsp;

## ✅ Execution

The script executes the following tasks sequentially while logging on. The scheduled task created by `Envoy-core.ps1` will execute these functions.

1. **Drive Mapping**: Calls `Deploy-DriveMappings` to manage network drives.
2. **Registry Key Management**: Calls `Deploy-RegistryKeys` to manage registry keys.
3. **Process execution**: Calls `Deploy-ProcessExecution` to manage process executions.
4. **File Actions**: Calls `Deploy-FileActions` to manage file actions 
5. **Printer mapping**: Calls `Deploy-PrinterMappings` to manage printer mappings.

&nbsp;

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

## ⏩ Contributing
Contributions are welcome! Please open an issue or reach out to me to discuss your ideas.