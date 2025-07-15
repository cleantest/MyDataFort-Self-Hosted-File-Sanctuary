*******************************************************
# MyDataFort: Self-Hosted File Sanctuary
## Overview

The server runs on a linux machine(laptop) to enable remote file accessibility over the internet.

## Features
- Secure Remote Access: Encrypted file transfers via WireGuard VPN
- Web-Based Management: Intuitive FileBrowser interface for file operations
- Dynamic DNS: DuckDNS for consistent domain access without static IP
- Lightweight OS: Lubuntu minimizes resource usage
- Cross-Platform: Accessible from any device (web/WireGuard client)

***Technologies Used
- Lubuntu	OS -- Lightweight Linux OS host
- DuckDNS -- Free dynamic DNS service
- FileBrowser-- Web-based file manager
- WireGuard	VPN -- for secure remote access
- Nginx (Optional) --	Reverse proxy for HTTPS

***Prerequisites
 - Hardware: x86/ARM machine (even Raspberry Pi)
            1GB+ RAM, 10GB+ storage
            
 - Network: Ports 80, 443 (HTTP/HTTPS), 51820 (WireGuard) forwarded
   Router with dynamic DNS support (for DuckDNS)
            
 - Accounts: DuckDNS subdomain registered
 - Internet connectivity on both server and client devices.

FULL SETUP GUIDE

****DUCKDNS CONFIGURATION

    sudo apt install curl
    mkdir ~/duckdns
    echo 'echo url="https://www.duckdns.org/update?domains=YOURDOMAIN&token=YOURTOKEN" | curl -k -o ~/duckdns/duck.log -K -' > ~/duckdns/duck.sh
    chmod +x ~/duckdns/duck.sh
    */5 * * * * ~/duckdns/duck.sh >/dev/null 2>&1  # Add to crontab
  
  ##Your subdomain will now automatically track changes in your router's IP


****WIREGUARD VPN SETUP

1.Install WireGuard:

    sudo apt update
    sudo apt install wireguard
    sudo mkdir -p /etc/wireguard
    cd /etc/wireguard

2.Generate server/client pair private and public keys:

    wg genkey | tee privatekey | wg pubkey > publickey
    wg genkey | tee client_privatekey | wg pubkey > client_publickey
  Create WireGuard Config (wg0.conf):

    sudo nano /etc/wireguard/wg0.conf
  Example wg0.conf(Server config file):
    
      [Interface]
      Address = 10.0.0.1/24
      ListenPort = 51820
      PrivateKey = <Server_Private_Key>
      
      PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o <Your_Network_Interface> -j MASQUERADE
      PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o <Your_Network_Interface> -j MASQUERADE
      #Replace <Your_Network_Interface> (often eth0 or enp3s0) from ip a command.
      
   #Configure Firewall:
    
    sudo ufw allow 51820/udp
  
   #Add WireGuard Client Peers:
    On client device, install wireguard and have this in the config file.
    
   NB//~ Copy the client's public key to the peer section of the server config file(wg0.conf) and the server's public key to the client's config file. 
    
    [Interface]
    PrivateKey = <Client_Private_Key>
    Address = 10.0.0.2/24
    DNS = 1.1.1.1
    
    [Peer]
    PublicKey = <Server_Public_Key>
    Endpoint = your-duckdns-subdomain.duckdns.org:51820
    AllowedIPs = 10.0.0.0/0 (Confines server traffic to only the vpn) (0.0.0.0/0 makes all traffic on the client pass through the VPN)
    PersistentKeepalive = 25

File Browser Setup(Web File Manager)

  Download & Install:
  
    curl -fsSL https://raw.githubusercontent.com/filebrowser/get/master/get.sh | bash
      
  Create a systemd Service: (This enables filebrowser to start automatically on boot)
  
      sudo nano /etc/systemd/system/filebrowser.service
  //Paste this
      
      [Unit]
      Description=File Browser
      After=network.target
      
      [Service]
      ExecStart=/usr/local/bin/filebrowser -r /home/your_user/desired_root_folder_to_share -p 8080
      User=your_user
      Restart=always
      
      [Install]
      WantedBy=multi-user.target
      
      
  ##Enable & Start File Browser:
  
      sudo systemctl enable filebrowser
      sudo systemctl start filebrowser
  ##Access File Browser on client device via:
  
    http://<LAN_IP or via VPN:
    http://10.0.0.1:8080

PORT FORWARDING AND REMOTE ACCESS

    Log into your router. 192.168.XX.1 
    Search for virtual settings and forward UDP 51820 to your server’s LAN IP (for WireGuard).

You now have secure remote access to your files from anywhere globally.

Congratulations and enjoy your extra storage space :)
