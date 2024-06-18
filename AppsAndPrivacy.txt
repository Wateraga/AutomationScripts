# Remove bloatware
Log-Message "Removing bloatware..."
$appsToRemove = @(
    "Microsoft.Microsoft3DViewer",
    "Microsoft.MicrosoftOfficeHub",
    "Microsoft.MicrosoftSolitaireCollection",
    "Microsoft.MicrosoftStickyNotes",
    "Microsoft.MixedReality.Portal",
    "Microsoft.OneConnect",
    "Microsoft.People",
    "Microsoft.Print3D",
    "Microsoft.SkypeApp",
    "Microsoft.Todos",
    "Microsoft.Wallet",
    "Microsoft.WindowsFeedbackHub",
    "Microsoft.XboxIdentityProvider",
    "Microsoft.XboxSpeechToTextOverlay",
    "Microsoft.YourPhone",
    "Microsoft.ZuneMusic",
    "Microsoft.ZuneVideo"
)

foreach ($app in $appsToRemove) {
    Get-AppxPackage -Name $app | Remove-AppxPackage -ErrorAction SilentlyContinue
    Get-AppxPackage -AllUsers -Name $app | Remove-AppxPackage -ErrorAction SilentlyContinue
}

# Set up privacy settings
Log-Message "Setting up privacy settings..."
# Disable advertising ID
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\AdvertisingInfo" -Name "Enabled" -Value 0

# Turn off tailored experiences
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Privacy" -Name "TailoredExperiencesWithDiagnosticDataEnabled" -Value 0

# Turn off location tracking
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\location" -Name "Value" -Value "Deny"

# Turn off background apps
$backgroundApps = Get-Process | Where-Object { $_.MainWindowTitle -eq "" } | Select-Object -ExpandProperty Name
foreach ($app in $backgroundApps) {
    Stop-Process -Name $app -Force -ErrorAction SilentlyContinue
}

# Disable Windows Hello
Set-ItemProperty HKLM:\SOFTWARE\Policies\Microsoft\Windows\System -Name 'AllowDomainPINLogon' -Value 0
Set-ItemProperty HKLM:\SOFTWARE\Microsoft\PolicyManager\default\Settings\AllowSignInOptions -Name 'value' -Value 0
New-Item -Path 'HKLM:\SOFTWARE\Policies\Microsoft\' -Name 'Biometrics' -Force
New-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Biometrics' -Name 'Enabled' -Value 0 -PropertyType Dword -Force
New-Item -Path 'HKLM:\SOFTWARE\Policies\Microsoft\' -Name 'PassportforWork' -Force
New-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\PassportforWork' -Name 'Enabled' -Value 0 -PropertyType Dword -Force
Start-Process cmd -ArgumentList '/s,/c,takeown /f C:\Windows\ServiceProfiles\LocalService\AppData\Local\Microsoft\NGC /r /d y & icacls C:\Windows\ServiceProfiles\LocalService\AppData\Local\Microsoft\NGC /grant administrators:F /t & RD /S /Q C:\Windows\ServiceProfiles\LocalService\AppData\Local\Microsoft\Ngc & MD C:\Windows\ServiceProfiles\LocalService\AppData\Local\Microsoft\Ngc & icacls C:\Windows\ServiceProfiles\LocalService\AppData\Local\Microsoft\Ngc /T /Q /C /RESET' -Verb runAs