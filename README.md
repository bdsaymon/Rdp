name: Setup RDP with Ngrok (Windows 11)

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: windows-latest
    timeout-minutes: 360

    steps:
    - name: Checkout Repository
      uses: actions/checkout@v3

    - name: Download ngrok
      run: |
        Invoke-WebRequest https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-windows-amd64.zip -OutFile ngrok.zip
        Expand-Archive ngrok.zip -DestinationPath .
        if (!(Test-Path .\ngrok.exe)) { throw "Ngrok download failed!" }

    - name: Authenticate ngrok
      run: .\ngrok.exe authtoken $env:NGROK_AUTH_TOKEN
      env:
        NGROK_AUTH_TOKEN: ${{ secrets.NGROK_AUTH_TOKEN }}

    - name: Enable Remote Desktop
      run: |
        Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 0
        Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
        Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "UserAuthentication" -Value 1

    - name: Set RDP User Password
      run: |
        $user = "rdpuser"
        $pass = ConvertTo-SecureString "Password123!" -AsPlainText -Force
        New-LocalUser $user -Password $pass
        Add-LocalGroupMember -Group "Administrators" -Member $user

    - name: Start ngrok tunnel (RDP)
      run: Start-Process -FilePath ".\ngrok.exe" -ArgumentList "tcp 3389"

    - name: Wait for tunnel and show address
      shell: pwsh
      run: |
        Start-Sleep -Seconds 15
        $tunnel = Invoke-RestMethod -Uri http://127.0.0.1:4040/api/tunnels
        $addr = $tunnel.tunnels[0].public_url
        Write-Output "✅ RDP Address (copy this): $addr"
        Write-Output "👉 Username: rdpuser"
        Write-Output "👉 Password: Password123!"

    - name: Keep alive (6 hours)
      run: |
        for ($i = 0; $i -lt 360; $i++) {
          Write-Output "RDP still running... Minute: $($i*1)"
          Start-Sleep -Seconds 60
        }
