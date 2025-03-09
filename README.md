# OfflinePiServer
Guide to creating a Private Offline Pi WIFI Network

## Requirements:
Raspberry PI (4B)
USB/SD CARD
ETHERNET CABLE

## PreReq:
1. Download Raspberry PI Imager (https://www.raspberrypi.com/software/)
2. After installed run and set up imager with custom settings (user/pass, wirelesslan, locale, ssh (password for simplicity))

## MAIN:
Installing Required Packages
1. On main PC SSH in while on same network ```ssh username@hostname.local```
2. In SSH run:
```
sudo apt update && sudo apt upgrade -y
sudo apt install -y hostapd dnsmasq iptables-persistent
```

## Removing Main WiFi Configuration
3. Since we set up WIFI with the imager we have to remove the settings from where the imager saves them. 
   Clear this entire file: ```/etc/wpa_supplicant/wpa_supplicant.conf```

## Configuring HostAPD (WiFi Access Point)
4. Create hostapd config file ```sudo nano /etc/hostapd/hostapd.conf```

Copy text into file:
```
interface=wlan0
driver=nl80211
ssid=testserv
hw_mode=g
channel=6
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=SecurePass123
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
```
7. Point Hostapd to new config file ```sudo nano /etc/default/hostapd```.
Edit ```DAEMON_CONF=""``` to include the correct path ```"/etc/hostapd/hostapd.conf"```

## Configuring dnsmasq (DHCP Server)
7. Backup default DNSMASQ config file ```sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig```
8. Create new DNSMASQ config file ```sudo nano /etc/dnsmasq.conf``` 

Copy text into file:
```
interface=wlan0
dhcp-range=192.168.2.10,192.168.2.100,24h
```

## Setting Up IP Forwarding & Firewall Rules
9. Edit file ```sudo nano /etc/sysctl.conf``` and uncomment line ```"net.ipv4.ip_forward=1"```.
Apply changes with ```sudo sysctl -p```
11. Set up NAT and forwarding rules:
```
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o wlan0 -j ACCEPT
```
12. Save even when restarted ```sudo netfilter-persistent save```

## Test for SSH (again)
12. Start and enable services ```sudo systemctl unmask hostapd```
```sudo systemctl enable hostapd dnsmasq```
```sudo systemctl start hostapd dnsmasq```
13. See if you can join local wifi network "testserv" on main PC
14. SSH in ```ssh username@hostname.local```

## Configure Permanent Static IP
15. Check if network manager is active ```sudo systemctl status NetworkManager```
16. Using SystemD create config file ```sudo nano /etc/systemd/network/10-wlan0.network```

Copy text into file:
```
[Match]
Name=wlan0

[Network]
Address=192.168.2.1/24
```
19. Enable and restart systemd-networkd ```sudo systemctl enable systemd-networkd```
```sudo systemctl restart systemd-networkd```
20. Reboot Pi
21. Confirm static IP used ```ip a show wlan0```

##RECAP:
1. RaspberryPi IP: 192.168.2.1
2. Network ID: testserv
3. The Pi is isolated from your main network and runs its own static IP configuration via systemd-networkd.
