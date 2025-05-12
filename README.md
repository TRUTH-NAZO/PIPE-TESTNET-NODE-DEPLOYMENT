# PIPE-TESTNET-NODE-DEPLOYMENT
# PIP TESTNET NODE DEPLOYMENT

This guide outlines step-by-step instructions for setting up and configuring the POP Cache Node on Linux systems using the provided binaries.

To be eligible for node rewards, you must be whitelisted and receive an email with setup instructions.
New users can sign up (https://airtable.com/apph9N7T0WlrPqnyc/pagSLmmUFNFbnKVZh/form), then wait for an email containing your invite code.


# System Requirements

Operating System: Linux (Ubuntu 24.04 only — earlier versions are not supported)

Memory: Minimum 16GB RAM (configurable; more RAM may increase reward potential)

Storage: SSD with at least 100GB of available space

Network: Stable 24/7 internet connectivity

If you've previously run a devnet node, follow these steps before proceeding.

# You will need a VPS, you can get from PQ hosting or Contabo. Buy your VPS from any of these two below
https://contabo.com https://pq.hosting/en/

# Backup Existing Node Info
Backup the old node_info.json file located in the pipe folder at your root directory.

        cp ~/pipe/node_info.json ~/node_info_backup.json

# Stop and Remove Old Service

          sudo systemctl stop pipe
          sudo systemctl disable pipe
          sudo rm /etc/systemd/system/pipe
          sudo systemctl daemon-reload

# Open Required Ports

          sudo ufw allow 80/tcp
          sudo ufw allow 443/tcp
          sudo ufw reload

# ⚙️ Node Setup

# Install Required Dependencies

          sudo apt update
          sudo apt install -y libssl-dev ca-certificates

# Configure System Settings
Update Kernel Parameters

          sudo bash -c 'cat > /etc/sysctl.d/99-popcache.conf << EOL
          net.ipv4.ip_local_port_range = 1024 65535
          net.core.somaxconn = 65535
          net.ipv4.tcp_low_latency = 1
          net.ipv4.tcp_fastopen = 3
          net.ipv4.tcp_slow_start_after_idle = 0
          net.ipv4.tcp_window_scaling = 1
          net.ipv4.tcp_wmem = 4096 65536 16777216
          net.ipv4.tcp_rmem = 4096 87380 16777216
          net.core.wmem_max = 16777216
          net.core.rmem_max = 16777216
          EOL'
          sudo sysctl -p /etc/sysctl.d/99-popcache.conf
          Set File Limits
          bash
          Copy
          Edit
          sudo bash -c 'cat > /etc/security/limits.d/popcache.conf << EOL
          *    hard nofile 65535
          *    soft nofile 65535
          EOL'

Note: Reboot or log out and back into your VPS after setting limits.

# Create Required Directories

        sudo mkdir -p /opt/popcache/logs
        
        cd /opt/popcache

# Download and Extract Pipe Binaries
Visit the provided download link (check your invite email).

# Use SFTP (e.g., Termius) to upload the file to /opt/popcache.

      sudo tar -xzf pop-v0.3.0-linux-*.tar.gz
      sudo chmod +x /opt/popcache/pop

# 🛠️ Configuration File
Create and edit config.json:

      nano /opt/popcache/config.json
      
# Paste and customize the following:


      {
        "pop_name": "your-pop-name",
        "pop_location": "Your Location, Country",
        "invite_code": "Enter your Invite Code",
        "server": {
          "host": "0.0.0.0",
          "port": 443,
          "http_port": 80,
          "workers": 0
        },
        "cache_config": {
          "memory_cache_size_mb": 4096,
          "disk_cache_path": "./cache",
          "disk_cache_size_gb": 100,
          "default_ttl_seconds": 86400,
          "respect_origin_headers": true,
          "max_cacheable_size_mb": 1024
        },
        "api_endpoints": {
          "base_url": "https://dataplane.pipenetwork.com"
        },
        "identity_config": {
          "node_name": "your-node-name",
          "name": "Your Name",
          "email": "your.email@example.com",
          "website": "https://your-website.com",
          "discord": "your_discord_username",
          "telegram": "your_telegram_handle",
          "solana_pubkey": "YOUR_SOLANA_WALLET_ADDRESS_FOR_REWARDS"
        }
      }

#🔎 Tips

 To check VPS timezone location:

        realpath --relative-to /usr/share/zoneinfo /etc/localtime

Website can be your GitHub profile or any personal webpage.

# 🔧 Create systemd Service

      sudo bash -c 'cat > /etc/systemd/system/popcache.service << EOL
      [Unit]
      Description=POP Cache Node
      After=network.target
      
      [Service]
      Type=simple
      User=root
      Group=root
      WorkingDirectory=/opt/popcache
      ExecStart=/opt/popcache/pop
      Restart=always
      RestartSec=5
      LimitNOFILE=65535
      StandardOutput=append:/opt/popcache/logs/stdout.log
      StandardError=append:/opt/popcache/logs/stderr.log
      Environment=POP_CONFIG_PATH=/opt/popcache/config.json
      
      [Install]
      WantedBy=multi-user.target
      EOL'

      sudo systemctl daemon-reload
      sudo systemctl enable popcache
      sudo systemctl start popcache


# 🩺 Node Health Checks
# Check Status

      sudo systemctl status popcache

# View Logs

      sudo journalctl -u popcache

# 🧰 Optional: Manage Node
# Stop the Node

      sudo systemctl stop popcache

# Restart the Node

      sudo systemctl daemon-reload
      sudo systemctl enable popcache
      sudo systemctl restart popcache

# Follow me on X and stay updated
Twitter - https://x.com/0x_nazo
