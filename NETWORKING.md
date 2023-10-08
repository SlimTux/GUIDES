# NETWORKING

## TOC
1. [SETUP](#setup)  
1.1 [/etc/network/interfaces](#/etc/network/interfaces)  
1.2. [WiFi](#wifi)  
2. [FIREWALL](#firewall)  
2.1. [ufw](#ufw)  
2.2. [iptables](#iptables)  
3. [SSH](#ssh)  
3.1. [CLIENT](#client)  
3.2. [SERVER](#server)  
4. [TROUBLESHOOTING](troubleshooting)  
4.1. [tcpdump](#tcpdump)  
4.2. [netstat](#netstat)  
4.3. [traceroute](#traceroute)  
4.4. [nmap](#nmap)  

## SETUP

### /etc/network/interfaces
```
# use last 8 octets for hosts
255.255.255.0
```  

### WiFi

Use WiFi without a separate network manager with this simple guide. Needs "_dhcpcd_" or "_dhcpclient_", "_net-tools_" or "_iproute2_", "_wpa\_supplicant_", and the WiFi drivers for your wireless card (like "_iwlwifi_" and its "_ucode_"), which in part can be installed from a package usually named "_linux-firmware_", but they may not be complete (this provides "_ucode_" but not "_iwlwifi_").  
__NOTE__: The "_<DEVICE_NAME>_" can be either "_wlp3s0_" or "_wlan0_". Change accordingly the following commands to suit your needs.

* Create the configuration file (as "_root_", not "_sudo_"):  
`wpa_passphrase <NETWORK_NAME> <PASSWORD> > /etc/wpa_supplicant.conf`  
* Delete non hashed password from "_/etc/wpa_supplicant.conf_", but not the hashed one.  

Each time you need to connect type the following command (as "_root_" or with "_sudo_"):  

* __EXAMPLE 1__: With "_net-tools_" and "_dhcpcd_":  
```  
ifconfig <DEVICE_NAME> down
ifconfig <DEVICE_NAME> up
wpa_supplicant -B -i<DEVICE_NAME> -c /etc/wpa_supplicant.conf -Dwext
dhcpcd <DEVICE_NAME>  
```

* __EXAMPLE 2__: With "_iproute2_" and "_dhclient_":  
```  
ip link set <DEVICE_NAME> down
ip link set <DEVICE_NAME> up
wpa_supplicant -B -i<DEVICE_NAME> -c /etc/wpa_supplicant.conf -Dwext
dhclient <DEVICE_NAME>  
```

You can save either example in a script to activate the Wi-Fi whenever you want.  

* Note: As an educational tip, the name of a network is also called "_SSID_" in other places.

## FIREWALL

### ufw
* Show status  
`sudo ufw status`  
* Enable firewall  
`sudo ufw enable`  
* Disable firewall  
`sudo ufw disable`  
* Deny all by default  
`sudo ufw default deny`  
* Allow all by default  
`sudo ufw default allow`  
* Allow everything for specific port by default  
`sudo ufw allow PORT_NUMBER`  
* Delete a rule  
`sudo ufw delete allow PORT_NUMBER`  
* Allow everything for a specific address  
`sudo ufw allow from IP_ADDRESS`  
* Allow a specific port for a specific address  
`sudo ufw allow from IP_ADDRESS to any port PORT_NUMBER`  

### iptables
* To list all rules:  
`iptables -L`  
* To flush all rules (reset to blank slate):  
`iptables -F`  
* To flush an specific rule:  
`iptables -D <THE_RULE_TO_FLUSH>`  

#### BASICS
* The rules are read in the order you give them and also their flags:  
_-A_: appends to previous list of rules.  
_-I_: inserts to previous list of rules.  

* The rules are followed according to their type which is a chain. The three types of chains are:  
_INPUT_: Comes from outside the firewall (commonly from another computer).  
_OUTPUT_: Comes from behind the firewall (commonly from the same computer).  
_FORWARD_: Goes to a third computer.  

* To select the interface (can be eth0, lo, wlan0, etc.):  
`--in-interface <INTERFACE>`  
* or also:  
`-i <INTERFACE>`  
* To make the rule match all but the requested interface add an exclamation between the interface flag and the interface name:  
`-i ! <INTERFACE>`  

* To select source of connection:  
`-s <SOURCE_IP>`  

* To select the protocol (can be tcp, udp, etc.):  
`-p <PROTOCOL>`  

* To select the port:  
`--dport <PORT>`  

* Match packet rules by state (can be used instead of ports):  
`-m state`  
* Types of state (ESTABLISHED, RELATED, etc.), more than one can be selected by using a comman with no spaces, for example:  
`--state ESTABLISHED,RELATED`  

* Match packet rules by IP range (can be used instead of ports):  
`-m iprange`  
* To choose a range set the start IP and the end IP separated by a dash:  
`--src-range <FIRST_IP>-<LAST_IP>`  

* The action to enforce (ACCEPT, DROP, etc.):  
`-j <ACTION>`  

#### GENERAL POLICIES
* Let pass all connections from inside the firewall:  
`iptables -P OUTPUT ACCEPT`  
* Drop all incoming connections by default:  
`iptables -P INPUT DROP`  
* Drop all forwarding connections by default:  
`iptables -P FORWARD DROP`  

* Allow all packets from loopback (your computer):  
`iptables -A INPUT --in-interface lo -j ACCEPT`  

* Allow connections from outisde to view your server:  
`iptables -A INPUT -p tcp --dport <SERVER_PORT> -j ACCEPT`  

* Allow connections to your computer through SSH (assuming the SSH server is running in port 22):  
`iptables -A INPUT -p tcp --dport 22 -j ACCEPT`  

* Allow SSH only from local IP using IP range (to be used instead of the above):  
`iptables -A INPUT -m iprange --src-range 192.168.1.1-192.168.1.254 -p tcp --dport 22 -j ACCEPT`  

* Allow connections to receive a response from the same port, for the sake of the two-way connection as in the case of web browsers:  
`iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT`  

* Drop spoofed packets simulating as coming from the same computer:  
`iptables -A INPUT --in-interface ! lo --source 127.0.0.0/8 -j DROP`  

#### CUSTOM POLICIES

* To create a custom chain:  
`-N <ANY_NAME>`  

* Declaring the <ANY_NAME> chain will add the deployment of rules with this chain name where this chain is called:  
`iptables -A INPUT -j <ANY_NAME>`  

* Using the chain <ANY_NAME> for connections from outisde to your server:  
`iptables -A <ANY_NAME> -p tcp --dport <SERVER_PORT> -j ACCEPT`  

* Using the chain <ANY_NAME> for connections to the SSH server:  
`iptables -A <ANY_NAME> -p tcp --dport 22 -j ACCEPT`  

#### PORT REDIRECTION
* Redirect port 80 to port 8080 using the NAT table:  
`iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080`  


## SSH
### CLIENT
* login to remote host  
`ssh ADDRESS`  
* login to remote host as user USER  
`ssh USER@ADDRESS`  

## SERVER
* set ssh server configuration in /etc/ssh/sshd_config  
```
Port 22 # default port is 22, can be changed
PermitRootLogin without-password # change "without-password" to "no" to forbid root login
AllowUsers USER_NAME # by allowing a specific user it restricts the others
```
* restart "ssh" service to activate changes  

## TROUBLESHOOTING

### tcpdump
* dump all  
`sudo tcpdump`  
* dump 5 packets  
`sudo tcpdump -c 5`  
* dump in ASCii format  
`sudo tcpdump -A`  
* dump in hexadecimal format  
`sudo tcpdump -xx`  
* dump from an specific interface  
`sudo tcpdump -i INTERFACE_NAME`  
* dump from a specific port  
`sudo tcpdump port PORT_NUMBER`  
* dump 5 packets in hexadecimal from an specific interface and a specific port  
`sudo tcpdump -c 5 -xx -i INTERFACE port PORT_NUMBER`  

### netstat
* show routing table, including gateway  
`netstat -nr`  
* show all ports  
`netstat -tulpn`  
* show network usage of devices  
`netstat -i`  
* show active connections  
`netstat -ta`  
* show active connections, but show ip addresses instead  
`netstat -tan`  

### traceroute
* show which route your connection takes between your computer to the destination  
`traceroute WEBNAME_OR_IP`  

### nmap
* scan a specific ip address (including devices)  
`nmap IP_NUMBER`  
* scan a specific website  
`nmap WEBSITE_NAME`  
* scan a specific ip address (including devices) with more information  
`nmap -v IP_NUMBER`  
* scan two ip address (including devices), 192.168.0.1 and 192.168.0.54  
`nmap 192.168.0.1,54`  
* scan a range of ip address (including devices), from 192.168.0.1 to 192.168.0.100  
`nmap 192.168.0.1-100`  
* scan all ip address (including devices) from network 192.168.0.0  
`nmap 192.168.0.*`  
* scan address from a file  
`nmap -il <FILE>`  
* scan address and identify OS and running services  
`nmap -A IP_NUMBER`  
* check if target is up  
`nmap -sP IP_NUMBER`  
* check reason for services states  
`nmap --reason IP_NUMBER`  
* show host interfaces  
`nmap --iflist IP_NUMBER`  
