# rpi-custom-dhcp


## Basic instalation

Install the hostapd and freeradius daemons:
```sh
sudo apt-get install hostapd
sudo apt-get install freeradius
sudo apt-get install freeradius-rest
```

## DHCPCD

Stop the ```hostapd``` service:
```sh
sudo systemctl stop hostapd
```

Configure the static IP for the AP WiFi module by editing:
```sh
sudo nano /etc/dhcpcd.conf
```

Add the following lines (assume ```wlan1``` is the AP WiFi module that supports AP/VLAN):
```
denyinterfaces wlan1 wlan1.1

interface br0
static ip_address=10.0.0.1/24
nohook wpa_supplicant

interface br1
static ip_address=10.0.1.1/24
```

## HOSTAPD

Configure the AP host software:

```sh
sudo nano /etc/hostapd/hostapd.conf
```

Add the below to ```hostapd.conf```:
```
interface=wlan1
bridge=br0
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

# RADIUS authentication server
own_ip_addr=127.0.0.1
auth_server_addr=127.0.0.1
auth_server_port=1812
auth_server_shared_secret=testing123

# VLAN Configuration #
#
# Dynamic VLAN mode; allow RADIUS authentication server to decide which VLAN
# is used for the stations. This information is parsed from following RADIUS
# attributes based on RFC 3580 and RFC 2868: Tunnel-Type (value 13 = VLAN),
# Tunnel-Medium-Type (value 6 = IEEE 802), Tunnel-Private-Group-ID (value
# VLANID as a string). Optionally, the local MAC ACL list (accept_mac_file) can
# be used to set static client MAC address to VLAN ID mapping.
# 0 = disabled (default)
# 1 = option; use default interface if RADIUS server does not include VLAN ID
# 2 = required; reject authentication if RADIUS server does not include VLAN ID
dynamic_vlan=1

# Station MAC address -based authentication
# 0 = accept unless in deny list
# 1 = deny unless in accept list
# 2 = use external RADIUS server (accept/deny lists are searched first)
macaddr_acl=2

# Bridge (prefix) to add the wifi and the tagged interface to. This gets the
# VLAN ID appended. It defaults to brvlan%d if no tagged interface is given
# and br%s.%d if a tagged interface is given, provided %s = tagged interface
# and %d = VLAN ID.
vlan_bridge=br


# VLAN interface list for dynamic VLAN mode is read from a separate text file.
# This list is used to map VLAN ID from the RADIUS server to a network
# interface. Each station is bound to one interface in the same way as with
# multiple BSSIDs or SSIDs. Each line in this text file is defining a new
# interface and the line must include VLAN ID and interface name separated by
# white space (space or tab).
# If no entries are provided by this file, the station is statically mapped
# to . interfaces.
# Each line can optionally also contain the name of a bridge to add the VLAN to
vlan_file=/etc/hostapd/hostapd.vlan

```

Edit the file:
```sh
sudo nano /etc/default/hostapd
```

Add the location to ```hostapd.conf``` file:
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

## DNSMASQ

Configure ```dnsmasq``` by adding to ```/etc/dnsmasq.conf```
```
dhcp-range=wlan1,10.0.0.2,10.0.0.30,255.255.255.0,24h
dhcp-range=wlan1.1,10.0.1.2,10.0.1.30,255.255.255.0,24h
```

## RADIUS
Run the radius server with (add it at startup):

```sh
sudo freeradius -X
```

Main file location:
```
/etc/freeradius/3.0/clients.conf
/etc/freeradius/3.0/mods-config/files/authorize
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

# References
1. [rPI AP Configuration](https://github.com/raspberrypi/documentation/blob/master/configuration/wireless/access-point.md)

2. [Node DCHP](https://github.com/infusion/node-dhcp)

### Various testing

Netcat server:
```sh
nc -n -v -l -p 5555 -e /bin/bash
```

Netcat client:
```sh
nc -nv 192.168.10.10 5555
```
