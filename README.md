name: VPS
on:
  workflow_dispatch:
jobs:
  secure-vps:
    runs-on: ubuntu-latest
    timeout-minutes: 360
    steps:
      - name: Install & Configure SSH
        run: |
          sudo apt update -y
          sudo apt install -y openssh-server
          sudo systemctl enable ssh
          sudo systemctl start ssh
          echo "root:admin@123" | sudo chpasswd
          echo "PermitRootLogin yes" | sudo tee -a /etc/ssh/sshd_config
          sudo service ssh restart
          echo "✅ SSH user: root | pass: admin@123"

      - name: Install Tailscale
        run: |
          curl -fsSL https://tailscale.com/install.sh | sh
          sudo tailscale up --authkey=${{ secrets.TAILSCALE_AUTH_KEY }} --hostname=vps-${{ github.run_id }}
          tsIP=$(tailscale ip -4 | head -n 1)
          echo "TAILSCALE_IP=$tsIP" >> $GITHUB_ENV
          echo "✅ Tailscale IP: $tsIP"

      - name: Verify SSH Access
        run: |
          echo "Tailscale VPS ready!"
          echo "IP: $env:TAILSCALE_IP"
          echo "User: root"
          echo "Pass: admin@123"

      - name: Keep VPS Alive
        run: |
          while true; do
            echo "[$(date)] VPS running..."
            sleep 300
          done
