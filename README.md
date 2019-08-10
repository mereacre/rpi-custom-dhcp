# rpi-custom-dhcp
rpi-custom-dhcp


## Basic instalation

Install the hostapd daemon:
```sh
sudo apt-get install hostapd
```

Stop the ```hostapd``` service:
```sh
sudo systemctl stop hostapd
```

Configure the static IP for the AP WiFi module by editing:
```sh
sudo nano /etc/dhcpcd.conf
```

Add the following lines (assume ```wlan0``` is the AP WiFi module):
```
interface wlan0
static ip_address=10.0.0.1/24
nohook wpa_supplicant
```

Configure the AP host software:

```sh
sudo nano /etc/hostapd/hostapd.conf
```

Add the below to ```hostapd.conf```:
```
interface=wlan0
bridge=br0
driver=nl80211
ssid=RPi_AP
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=1234554321
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

Edit the file:
```sh
sudo nano /etc/default/hostapd
```

Add the location to ```hostapd.conf``` file:
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

## Service restart

Restart services:
```sh
sudo service dhcpcd restart
sudo systemctl reload dnsmasq
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo systemctl start hostapd
```

Status finder:
```sh
sudo systemctl status hostapd
sudo systemctl status dnsmasq
```

## DHCP server configuration
Spread the broadcast to the ```wlan0``` interface:
```sh
sudo route add -host 255.255.255.255 dev wlan0
```
# References
1. [rPI AP Configuration](https://github.com/raspberrypi/documentation/blob/master/configuration/wireless/access-point.md)

2. [Node DCHP](https://github.com/infusion/node-dhcp)