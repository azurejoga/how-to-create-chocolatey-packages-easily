# how to create chocolatey packages easily
 create chocolatey packages quickly and easily for your application or program

# Chocolatey Package Creation Guide

## Introduction
This guide provides a step-by-step tutorial on creating Chocolatey packages to simplify software installation on your system. It covers all the steps in an easy-to-follow manner for users.

The motivation behind creating this guide/manual is because the Chocolatey documentation seems confusing to me. So, I decided to study it on my own and help other users who might be confused or want an easy and better way with examples!

## Prerequisites
Make sure Chocolatey is installed on your system. If not, you can install it by following the instructions below.

## Installing Chocolatey
To install Chocolatey in PowerShell, follow these steps:

1. Open PowerShell as an administrator.
2. Run the following command to download and install Chocolatey:

   ```powershell
   Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
   ```

   This command temporarily adjusts the execution policy, downloads the Chocolatey installation script from the web, and executes it.

3. Wait for the installation process to complete. It may take a few minutes depending on your internet connection.
4. After installation, close and reopen PowerShell.
5. To check if the installation was successful, run the following command:

   ```powershell
   choco --version
   ```

   If the installation is successful, you will see the current version of Chocolatey installed on your system.

## Step 1: Package Structure
A Chocolatey package consists of an organized set of files. Create a basic structure for your package:

```folder
MyPackage/
|-- MyPackage.nuspec
|-- tools/
|   |-- chocolateyInstall.ps1
|   |-- chocolateyUninstall.ps1
```

## Step 2: chocolateyInstall.ps1
Inside your `tools` folder, create the `chocolateyInstall.ps1` script responsible for installing the software. Example:

```powershell
$ErrorActionPreference = 'Stop'

$packageName = 'YourPackage'
$url = 'YourDirectLinkForSetupOrOther'
$checksum = 'YourChecksum'

# Download the .exe file using Chocolatey's built-in command
$downloadedFile = Get-ChocolateyWebFile -PackageName $packageName -FileFullPath "$toolsDir\YourPackage.exe" -Url $url -Checksum $checksum -ChecksumType 'sha256'

# Run the installer without waiting for completion
Start-Process -FilePath $downloadedFile

# Display a message to install manually
Write-Host "Please install manually. After installation, close this window."

# Wait for 1 second to give time for the installer to start
Start-Sleep -Seconds 1

# Exit the script
Exit
```

**Note:** If your installer or uninstaller is not silent, remove the silent arguments. 

PS: you can modify the script to your liking, this is just a basic example

## Step 3: chocolateyUninstall.ps1
Create the `chocolateyUninstall.ps1` script to uninstall the software if needed. Example:

```powershell
$packageName = 'YourPackage'
$installerType = 'EXE'
$silentArgs = '/S'
$validExitCodes = @(0)

Uninstall-ChocolateyPackage -PackageName $packageName -FileType $installerType -SilentArgs $silentArgs -ValidExitCodes $validExitCodes
```

## Create a .nuspec File
After creating the PowerShell scripts that contain both the installation and uninstallation methods for your application, create a .nuspec file containing all the project information.

Here's an example .nuspec file. Copy and paste, then rewrite your information:

```xml
<?xml version="1.0"?>
<package>
  <metadata>
    <id>YourPackageNameWhenPackaging</id>
    <version>YourVersion, e.g., 1.0, 2.0, 3.0</version>
    <title>NameOfTheApp</title>
    <authors>YourName</authors>
    <owners>YourNameOrNickname</owners>
    <description>
      Description about your app
    </description>
    <tags>TagsSeparatedWithSpaces</tags>
    <summary>
      Your summary
    </summary>
    <projectUrl>YourLinkOfThePackage</projectUrl>
    <iconUrl>DirectLinkForImage</iconUrl>
    <licenseUrl>YourLicence</licenseUrl>
    <packageSourceUrl>YourUrl</packageSourceUrl>
    <releaseNotes>YourURL</releaseNotes>
  </metadata>
  <files>
    <file src="chocolateyinstall.ps1" target="tools" />
  </files>
</package>
```

## Information
After completing these steps, you need to create a .NUPKG file to package your package for Chocolatey repository.

## Step 4: Create the Package
Run the following command in the root of your package to create the `.nupkg` file:

```powershell
choco pack
```

## Step 5: Test the Package
Before distribution, test the package locally:

```powershell
choco install YourPackage -dv -s .
```

Replace `YourPackage` with the name of your package in all interactions.

## Sending to the Repository
Let's go through the steps to send it to the repository.

1. Register for a Chocolatey account at [Chocolatey Community](https://community.chocolatey.org/account/Register).
2. Log in to your Chocolatey account at [Chocolatey](https://community.chocolatey.org/users/account/LogOn).
3. Get your API key: apikey.
4. After obtaining your API key, use

 the following command in PowerShell within the directory where your .nupkg file is located:

   ```powershell
   choco apikey --key YourApiKey -source https://push.chocolatey.org/
   ```

   The output displayed in your terminal will be:

   ```
   Registered API key with https://push.chocolatey.org/
   ```

5. After completing all of this, you can submit your package for automated review, automated testing, and moderation verification.

6. The command is:

   ```powershell
   choco push
   ```

   If there is more than one NUPKG file in the directory, you can specify the package with the following command:

   ```powershell
   choco push YOUR_PACKAGE_NAME.nupkg --source https://push.chocolatey.org/
   ```

7. The terminal output will be:

   ```
   Pushing MyPackage.1.0.nupkg to 'https://push.chocolatey.org/'
   ...
   Your package was pushed.
   Pushing MyPackage 1.0... 100%

   Chocolatey v0.10.15
```

## Warnings, Errors, and Tips
Sometimes errors may occur; here are the most common ones:

- **Invalid API Key:**
  - Error: `The API Key is invalid.`
  - Cause: The API key provided in the `choco push` command may be invalid.

- **Package Already Exists:**
  - Error: `A package version XXX already exists, refusing to overwrite. Use --force to force.`
  - Cause: There is already a version of the package with the same number in the repository. Use `--force` if you want to overwrite.

- **Package File Issue:**
  - Error: `Unable to find package XXX with version YYY.`
  - Cause: The package file (`.nupkg`) may be missing or damaged.

- **Connection Issues:**
  - Error: `Unable to process request. No connection could be made because the target machine actively refused it...`
  - Cause: Connectivity issues with the Chocolatey server.

- **Insufficient Permissions:**
  - Error: `Access to the path 'XXX' is denied.`
  - Cause: The user does not have sufficient permissions to access or write to the package directory. Or the ID <id> is equal to the package provided earlier.

- **Authentication Issues:**
  - Error: `Unauthorized - The user does not have required permissions.`
  - Cause: The provided API key may not have sufficient permissions to push packages.

- **Issues with Push Command:**
  - Error: `choco : The term 'choco' is not recognized as the name of a cmdlet...`
  - Cause: Chocolatey is not installed or not in your system path.

You can find solutions in the [Chocolatey documentation on troubleshooting](https://docs.chocolatey.org/en-us/troubleshooting).

Additionally, you can track the progress of sending your packages on your Chocolatey moderation page or via email, where Chocolatey will send you status updates for your package.

## Useful Links
- [Create PowerShell Scripts](https://learn.microsoft.com/en-us/training/modules/introduction-to-powershell/)
- [Source of Information (Besides My Own Experience in Creating Chocolatey Packages)](https://docs.chocolatey.org/en-us/create/create-packages-quick-start)

## Conclusion
Now you know how to:
1. Create Chocolatey packages easily
2. Send Chocolatey packages
3. Create an account, log in, and get your API key.
4. Create a basic PowerShell script with Chocolatey methods.

I hope everyone enjoys this guide! 💙

tags

how to create packages
 create chocolatey packages
  create chocolatey app