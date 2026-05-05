name: RDP-High-Performance

on:
  workflow_dispatch:

jobs:
  secure-rdp:
    runs-on: windows-latest
    timeout-minutes: 3600

    steps:
      - name: Maximize System Performance
        run: |
          powercfg /setactive 8c5e7fda-e8bf-4a96-9a85-a6e23a8c635c
          reg add "HKCU\Control Panel\Desktop" /v UserPreferencesMask /t REG_BINARY /d 9012038010000000 /f
          reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\VisualEffects" /v VisualFXSetting /t REG_DWORD /d 2 /f

      - name: Configure Core RDP Settings
        run: |
          Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0 -Force
          Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name "UserAuthentication" -Value 0 -Force
          netsh advfirewall firewall add rule name="RDP-Port" dir=in action=allow protocol=TCP localport=3389
          Restart-Service -Name TermService -Force

      - name: Create RDP User (Complexity Bypass)
        run: |
          secedit /export /cfg config.inf
          (Get-Content config.inf) -replace 'PasswordComplexity = 1', 'PasswordComplexity = 0' | Set-Content config.inf
          (Get-Content config.inf) -replace 'MinimumPasswordLength = .+', 'MinimumPasswordLength = 0' | Set-Content config.inf
          secedit /configure /db $env:windir\security\local.sdb /cfg config.inf /areas SECURITYPOLICY
          
          $password = "Ender"
          $securePass = ConvertTo-SecureString $password -AsPlainText -Force
          New-LocalUser -Name "RDP" -Password $securePass -AccountNeverExpires
          Add-LocalGroupMember -Group "Administrators" -Member "RDP"
          Add-LocalGroupMember -Group "Remote Desktop Users" -Member "RDP"
          echo "TAILSCALE_PASS=$password" >> $env:GITHUB_ENV

      - name: Install and Connect Tailscale
        run: |
          Invoke-WebRequest -Uri "https://pkgs.tailscale.com/stable/tailscale-setup-latest-amd64.msi" -OutFile "tailscale.msi"
          Start-Process msiexec.exe -ArgumentList "/i tailscale.msi /qn /norestart" -Wait
          
          # Wait a moment for the service to initialize
          Start-Sleep -Seconds 5
          
          # Use the Secret Key to log in automatically
          & "C:\Program Files\Tailscale\tailscale.exe" up --authkey=${{ secrets.TAILSCALE_AUTH_KEY }} --hostname=gh-highperf-${{ github.run_id }}
          
          $tsIP = (& "C:\Program Files\Tailscale\tailscale.exe" ip -4)
          echo "TAILSCALE_IP=$tsIP" >> $env:GITHUB_ENV

      - name: Maintain Connection
        run: |
          Write-Host "`n=== RDP ACCESS DETAILS ==="
          Write-Host "Address: $env:TAILSCALE_IP"
          Write-Host "Username: RDP"
          Write-Host "Password: $env:TAILSCALE_PASS"
          Write-Host "==========================`n"
          
          while ($true) {
              Write-Host "[$(Get-Date)] Runner Active - Use RDP to connect to $env:TAILSCALE_IP"
              Start-Sleep -Seconds 300
          }
