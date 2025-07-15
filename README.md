# Remote-server
Remote server on a linux machine to access local files remotely from anywhere.

DUCKDNS 
(Dynamic IP Updater)
Create a duckdns account
Get this by querying, 'whatismyip' on the browser
After signing up on Duckdns, create a subdomain and enter your router's public IP

Install duckdns updater script
mkdir -p ~/duckdns
cd ~/duckdns
nano duck.sh

Paste the following script (replace with YOUR_DOMAIN and YOUR_TOKEN)
echo "url=https://www.duckdns.org/update?domains=YOUR_DOMAIN&token=YOUR_TOKEN&ip=" | curl -k -o ~/duckdns/duck.log -K -

Make Script Executable:
chmod 700 duck.sh

Automate with Cron:
crontab -e
*/5 * * * * ~/duckdns/duck.sh >/dev/null 2>&1

Your subdomain will now automatically track changes in your router's IP


WIREGUARD SETUP
1.Install WireGuard:

sudo apt update
sudo apt install wireguard

Configure WireGuard Server:
sudo mkdir -p /etc/wireguard
cd /etc/wireguard

Generate server private and public keys:

wg genkey | tee privatekey | wg pubkey > publickey
Create WireGuard Config (wg0.conf):

sudo nano /etc/wireguard/wg0.conf
Example wg0.conf(config):

[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <Server_Private_Key>

PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o <Your_Network_Interface> -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o <Your_Network_Interface> -j MASQUERADE
#Replace <Your_Network_Interface> (often eth0 or enp3s0) from ip a command.

Configure Firewall:
sudo ufw allow 51820/udp

2.Enable & Start WireGuard:
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0


Add WireGuard Client Peers:
Generate client keys:

wg genkey | tee client_privatekey | wg pubkey > client_publickey
Edit /etc/wireguard/wg0.conf:

[Peer]
PublicKey = <Client_Public_Key>
AllowedIPs = 10.0.0.2/32 (client's ip on vpn network)
Share client config:

[Interface]
PrivateKey = <Client_Private_Key>
Address = 10.0.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = <Server_Public_Key>
Endpoint = your-duckdns-subdomain.duckdns.org:51820
AllowedIPs = 10.0.0.0/0 (Confines server traffic to only the vpn) (0.0.0.0/0 makes all traffic on the client pass through the VPN)
PersistentKeepalive = 25

3.Install File Browser (Web File Manager)
Download & Install:
curl -fsSL https://raw.githubusercontent.com/filebrowser/get/master/get.sh | bash


