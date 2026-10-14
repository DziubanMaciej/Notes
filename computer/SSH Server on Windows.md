parentTopic:: [[Computer]]

## Install and enable
```powershell
# Install SSH server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# Check if it's installed
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH.Server*'

# Allow in firewall
New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
Set-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -Profile Any -Enabled True

# Start service
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'
```

## (Optional) Setup authorized keys
By default the file with accepted authorized keys is moved from `~/.ssh/authorized_keys` to `C:\ProgramData\ssh\administrators_authorized_keys`.

Open `C:\ProgramData\ssh\sshd_config` and comment out these lines
```
#Match Group administrators
#       AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```

Then run
```powershell
New-Item -ItemType File -Path "$env:USERPROFILE\.ssh\authorized_keys" -ErrorAction SilentlyContinue | Out-Null
icacls.exe "$env:USERPROFILE\.ssh\authorized_keys" /inheritance:r /grant "$($env:USERNAME):F" "SYSTEM:F" "BUILTIN\Administrators:F"
Restart-Service sshd
```

Do **not** create the file with `echo PUBLIC_KEY > ~/.ssh/authorized_keys`. It will write UTF-16LE encoding, which won't work.
