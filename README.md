# Windows False-Positive Detection Repair

A PowerShell utility for troubleshooting false-positive malware detection issues involving Google Chrome and Microsoft Defender.

## Features

- Updates Microsoft Defender security signatures.
- Checks Windows Defender protection status.
- Cleans selected Google Chrome cache directories.
- Displays Microsoft Defender threat history.
- Initiates a Microsoft Defender quick scan.
- Opens Windows Security.
- Does not disable antivirus or real-time protection.

## Requirements

- Windows 10 or Windows 11.
- Windows PowerShell 5.1 or later.
- Administrator privileges.
- Microsoft Defender Antivirus.

## Installation & Usage

1. Download or clone this repository.
2. Open PowerShell as Administrator.
3. Navigate to the downloaded project folder.
4. Run:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force
.\false-positive-repair.ps1
```

## Complete PowerShell Script

Save the following code as `false-positive-repair.ps1`.

```powershell
# ==========================================================
# WINDOWS FALSE-POSITIVE DETECTION REPAIR
# Chrome + Microsoft Defender
# ==========================================================

$ErrorActionPreference = "Continue"

Write-Host "`n=== FALSE POSITIVE REPAIR ===" -ForegroundColor Cyan

# 1. Check administrator privileges
$isAdmin = ([Security.Principal.WindowsPrincipal] `
    [Security.Principal.WindowsIdentity]::GetCurrent()
).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)

if (-not $isAdmin) {
    Write-Host "Please run PowerShell as Administrator." -ForegroundColor Red
    exit
}

# 2. Update Microsoft Defender signatures
Write-Host "`n[1/6] Updating Defender signatures..." -ForegroundColor Yellow

try {
    Update-MpSignature
    Write-Host "Defender signatures updated." -ForegroundColor Green
} catch {
    Write-Warning $_
}

# 3. Check Defender protection status
Write-Host "`n[2/6] Checking Defender protection..." -ForegroundColor Yellow

Get-MpComputerStatus |
    Select-Object AntivirusEnabled,
                  RealTimeProtectionEnabled,
                  AntivirusSignatureVersion,
                  AntivirusSignatureLastUpdated |
    Format-List

# 4. Reset Chrome download metadata/cache
Write-Host "`n[3/6] Cleaning Chrome download cache..." -ForegroundColor Yellow

$chromePaths = @(
    "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Cache",
    "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Code Cache",
    "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\GPUCache"
)

$chromeProcesses = Get-Process chrome -ErrorAction SilentlyContinue

if ($chromeProcesses) {
    Write-Host "Close Chrome completely before cleaning its cache." -ForegroundColor Red
} else {
    foreach ($path in $chromePaths) {
        if (Test-Path $path) {
            Remove-Item "$path\*" -Recurse -Force -ErrorAction SilentlyContinue
        }
    }
    Write-Host "Chrome cache cleanup completed." -ForegroundColor Green
}

# 5. Check Windows Security threat history
Write-Host "`n[4/6] Checking Defender threat history..." -ForegroundColor Yellow

try {
    Get-MpThreatDetection |
        Select-Object InitialDetectionTime,
                      ThreatID,
                      ActionSuccess,
                      Resources |
        Format-List
} catch {
    Write-Warning "Could not retrieve threat history."
}

# 6. Run Defender quick scan
Write-Host "`n[5/6] Running Microsoft Defender quick scan..." -ForegroundColor Yellow

try {
    Start-MpScan -ScanType QuickScan
    Write-Host "Quick scan initiated." -ForegroundColor Green
} catch {
    Write-Warning $_
}

# 7. Open Windows Security
Write-Host "`n[6/6] Opening Windows Security..." -ForegroundColor Yellow

Start-Process "windowsdefender:"

Write-Host "`n========================================" -ForegroundColor Cyan
Write-Host "REPAIR PROCEDURE COMPLETED" -ForegroundColor Green
Write-Host "Review Windows Security for any remaining detections."
Write-Host "========================================" -ForegroundColor Cyan
```

## Important Notes

- Close Google Chrome before running the script if you want its cache cleaned.
- The script does not delete Defender quarantine records.
- The script does not disable Windows Defender, SmartScreen, or real-time protection.
- Clearing browser cache does not reset Google Safe Browsing reputation.
- A successful run does not guarantee that a flagged file is safe or that future warnings will disappear.

## Investigating False Positives

Check Microsoft Defender detection records:

```powershell
Get-MpThreatDetection | Format-List *
```

Check Defender protection status:

```powershell
Get-MpComputerStatus | Format-List *
```

Check an application's digital signature:

```powershell
Get-AuthenticodeSignature "C:\Path\To\Application.exe"
```

## Official Reporting Resources

- [Microsoft Security Intelligence – File Submission](https://www.microsoft.com/en-us/wdsi/filesubmission)
- [VirusTotal](https://www.virustotal.com/)
- [Google Safe Browsing](https://safebrowsing.google.com/safebrowsing/report_error/)

## Disclaimer

This project is intended for legitimate Windows troubleshooting and false-positive investigation.

It does not bypass antivirus protection or guarantee that detected files are harmless. Always investigate security warnings before executing downloaded files.

## License

MIT License

Copyright (c) 2026

See [MIT License](https://opensource.org/license/mit) for details.
