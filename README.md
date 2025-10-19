name: Setup ngrok and Remote Desktop

on:
  workflow_dispatch:

jobs:
  setup-ngrok:
    runs-on: windows-latest

    steps:
      - name: Download ngrok
        run: |
          Invoke-WebRequest https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-windows-amd64.zip -OutFile ngrok.zip
          Expand-Archive ngrok.zip
          .\ngrok\ngrok.exe authtoken ${{ secrets.NGROK_AUTH_TOKEN }}

      - name: Enable RDP Access
        run: |
          Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\' -name "fDenyTSConnections" -Value 0
          Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
          Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp\' -name "UserAuthentication" -Value 1

      - name: Start ngrok tunnel
        run: Start-Process -NoNewWindow -FilePath .\ngrok\ngrok.exe -ArgumentList "tcp 3389"

      - name: Display RDP Information
        run: |
          Write-Output "✅ RDP server is running!"
          Write-Output "Use your ngrok dashboard to find the forwarding address."
