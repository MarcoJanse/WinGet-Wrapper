# WinGet-Wrapper

PowerShell Scripts to deploy and bulk import WinGet packages to InTune including metadata.<br>
Automatically detect latest version using dynamic detection script. <br>
Detection script checks local installed version against latest winget available version or a defined fixed target version.<br>
Bulk import WinGet packages to InTune including WinGet package metadata using WinGet-WrapperImportGUI.exe <br>
<br>

- Dynamically finds the WinGet directory to be used under System Context.
- Kill selected process before WinGet command
- Allows running pre and post script before installation
- Detection script that dynamically finds latest package available trough WinGet
- Requirement script to allow creating packages for update purposes only
- Logs to $env:ProgramData\WinGet-WrapperLogs (Usually C:\ProgramData\WinGet-WrapperLogs)
- Dynamically detect if running in user or system context
- Performs automatic cleanup of log files older than 60 days.
- Directly import and deploy WinGet packages to InTune including WinGet package metadata

## Background / Why?

WinGet have a few limitations in terms of automation and is not integrated with common endpoints management products.<br>
System Context is not possible by using "winget" as the .exe location must be found and this location is not static due to versioning in the directory name.<br>

## Requirements

- Windows 10 20H2 or newer.
- Powershell 5.1
- Client language must be en-US, as Winget-Wrapper parses only English output.
- Module "IntuneWin32App" and "Microsoft.Graph.Intune" needed for import to InTune.

## WinGet-WrapperImportGUI.exe

WinGet-WrapperImportGUI is a graphical interface designed to streamline the import of WinGet packages into InTune.<br>
This tool complements WinGet-Wrapper, providing an intuitive way to upload WinGet packages to InTune, along with their metadata.<br>

#### Features:

- **Search and Select:** Seamlessly search for WinGet packages, get detailed info and package versions.
- **InTune Integration:** Import selected WinGet packages directly into InTune for deployment.
- **CSV Support:** Export and import packages using CSV files, facilitating batch operations.<br>

![image](Images/WinGet-WrapperImportGUI.png)

#### Usage

>- **Open the GUI:** Run WinGet-WrapperImportGUI.exe to open the GUI
>- **Search Packages:** Enter your search query and click "Search" to find WinGet packages.
>- **Select Packages:** Select from search results, then click the center arrow to move them to the import list.
>- **Adjust:** Select target version if required, UpdateOnly, Installation context, etc.
>- **Import to InTune:** Enter your Tenant name (like `company.onmicrosoft.com`) in the field and click "Import to InTune" to import selected packages.
>- **Additional Actions:** Use buttons for exporting CSV, deleting, or importing from CSV.

## WinGet-Wrapper.ps1

![image](https://github.com/SorenLundt/WinGet-Wrapper/assets/127216441/4220b44b-7f96-4fb1-84ec-ce416f6f622c)

#### Usage

```posh
Powershell.exe -NoLogo -NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -File WinGet-Wrapper.ps1 -PackageName "PackageName for log file" -StopProcess "kill process using Stop-Process (do not add .exe)" -PreScript "somefile.ps1" -PostScript "somefile.ps1" -ArgumentList "Arguments Passed to WinGet.exe"
```

## WinGet-WrapperDetection.ps1

Matches locally installed version with newest available version using WinGet or specified version using $TargetVersion<br>
Can be setup to accept newer installed version locally $AcceptNewerVersion<br>

![image](https://github.com/SorenLundt/WinGet-Wrapper/assets/127216441/631d6001-b813-4b79-a12f-3c1e06cb3aec)

## WinGet-WrapperRequirements.ps1

Checks if application is detected locally. If not detected will not attempt update/install<br>
To be used when only wanting to update if application is already installed. (Update Only)<br><br>
![image](https://github.com/SorenLundt/WinGet-Wrapper/assets/127216441/b5af0ddd-6700-46cf-8907-33dbd0f8e930)

Outputs either "Installed" or "Not Installed"<br><br>

![image](https://github.com/SorenLundt/WinGet-Wrapper/assets/127216441/b8cd24fd-da34-4e1c-aeb2-0627717e1244)

## WinGet-WrapperImportFromCSV.ps1

Imports packages from WinGet to InTune (including available WinGet package metadata)<br>
Package content is stored under `Packages\Package.ID-Context-UpdateOnly-UserName-yyyy-mm-dd-hhssmm`<br>
Create deployment using csv columns: InstallIntent, Notification, GroupID<br><br>

![image](https://github.com/SorenLundt/WinGet-Wrapper/assets/127216441/dde433b9-81e1-484b-8ee1-71ac02d68441)<br>

## Usage: Import from CSV (Intune)

Open the sample CSV file WinGet-WrapperImportFromCSV.csv and add any WinGet Package IDs to import (Case Sensitive)<br>

### Usage

```powershell
WinGet-WrapperImportFromCSV.ps1 -TenantID company.onmicrosoft.com -csvFile WinGet-WrapperImportFromCSV.csv -SkipConfirmation
```

#### Process

![image](https://github.com/SorenLundt/WinGet-Wrapper/assets/127216441/297ddb07-eeac-41c7-a9ec-9656727984f6)<br>

#### Results

![image](https://github.com/SorenLundt/WinGet-Wrapper/assets/127216441/ca57c9d4-0ec7-4514-8694-7160f6356b5e)

#### Columns

- PackageID = Exact PackageID (Required)
- Context = Which context the Win32App is run under (Machine or User) (Required)
- AcceptNewerVersion = Allows newer installed version locally than specified (Set to 0 or 1)(Required)
- UpdateOnly = Update package only. Application will only update if application is already installed (Set to 0 or 1)(Required)
- TargetVersion = Specific version of the application. If not set, the package will always be the latest version
- StopProcessInstall = Kill a specific process (Stop-process) before installation (.exe should not be defined)
- StopProcessUninstall = Kill a specific process (Stop-process) before uninstallation (.exe should not be defined)
- PreScriptInstall = Run powershell script before installation
- PostScript = Run powershell script after installation
- PreScriptUninstall = Run powershell script before uninstallation
- PostScriptUninstall = Run powershell script after uninstallation
- CustomArgumentListInstall = Arguments passed to WinGet (default: `install --exact --id PackageID --silent --accept-package-agreements --accept-source-agreements --scope Context`)
- CustomArgumentListUninstall = Arguments passed to WinGet (default: `uninstall --exact --id PackageID --silent --accept-source-agreements --scope Context`)
- InstallIntent = Available or Required deployment.
- Notification = Notification level on deployment - Valid values: showAll, showReboot, hideAll.
- GroupID = InTune GroupID to deploy package to.

## Usage: Manual Import (InTune)

### Application Installation

In InTune create an Windows app (Win32) and upload WinGet-Wrapper.InTuneWin as the package file.  <br>
>**Install:** Powershell.exe -NoLogo -NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -File WinGet-Wrapper.ps1 -PackageName "VideoLAN.VLC" -StopProcess "VLC" -ArgumentList "install --exact --id VideoLAN.VLC --silent --accept-package-agreements --accept-source-agreements --scope machine"

>**Uninstall:** Powershell.exe -NoLogo -NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -File WinGet-Wrapper.ps1 -PackageName "VideoLAN.VLC" -StopProcess "VLC" -ArgumentList "Uninstall --exact --id VideoLAN.VLC --silent --accept-source-agreements --scope machine"

Change the $id variable to match the package id in the detection script and upload it  ($id = "VideoLAN.VLC")  <br>
  *If specific version is required change the $TargetVersion (Ex. $TargetVersion = "1.0.0.0")*  <br><br>
![image](https://github.com/SorenLundt/WinGet-Wrapper/assets/127216441/2aea611c-7733-4f93-9cbe-a44b4f66333d)

### Application Update Only

For creating application that will only update/install if application is already installed.<br>
Perform the same steps as in "Application Installation".<br>
Setup Requirement rule script with return string value of "Installed".<br><br>

![image](https://github.com/SorenLundt/WinGet-Wrapper/assets/127216441/b2bdb617-c74a-4902-9c2c-b8defe1adc70)

![image](https://github.com/SorenLundt/WinGet-Wrapper/assets/127216441/b8cd24fd-da34-4e1c-aeb2-0627717e1244)

## Is Winget Safe to Use in Enterprise?  <br>

Winget (Windows Package Manager) is generally safe for enterprise use due to the following security features: <br>

- Package Verification: Utilizes hash checks to ensure package integrity.
- Microsoft Vetting: Packages undergo thorough testing and scanning before approval.
- Manual Approvals: Human oversight adds an extra layer of security.
- User Controls: Enterprises can restrict installations through whitelisting.
- Regular Updates: Active maintenance and community involvement enhance security.
<br>

Overall, Winget is a secure option for enterprises, especially when proper management practices are implemented. Continuous monitoring is key to maintaining security.  <br>

## Disclaimer

While Winget provides various security measures, no software management tool is entirely risk-free. Organizations should continually assess their security posture and policies when using Winget or any other software deployment tool. <br>
<br>

**This software is provided "AS IS" with no warranties. Use at your own risk.** <br>
