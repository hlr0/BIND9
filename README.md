# BIND9 Complete DNS Server Configuration with Hostname - Step by Step Guide

This guide walks you through the process of setting up a complete DNS server on Ubuntu Server 12 using BIND9, with hostname configuration.

## Prerequisites

- Ubuntu Server
- Root or sudo privileges

## Step 1: Install BIND9

First, update the system and install BIND9 along with necessary utilities.


sudo apt update -y
sudo firewall-cmd --permanent --add-service=dns
sudo firewall-cmd --permanent --add-port=53/udp
sudo firewall-cmd --reload
sudo dnf install bind9 bind9-utils -y
sudo systemctl start named  # for systemd
sudo systemctl enable named
sudo systemctl status named  # Should show active
Step 2: Change IP Address from DHCP to Static
Edit the network interfaces file to set a static IP address.


sudo nano /etc/network/interfaces
Add the following configuration:


auto eth0
iface eth0 inet static
address 192.168.1.5
netmask 255.255.255.0
network 192.168.1.0
broadcast 192.168.1.255
gateway 192.168.1.1
# dns-nameservers (commented out, will configure later)
Restart networking daemons:


sudo /etc/init.d/networking restart
Step 3: Configure Hostname and Domain Name
Check the current hostname:


sudo nano /etc/hostname
Example:


nefitari
Now configure the domain name for your server:


sudo nano /etc/hosts
Example:


127.0.0.1 localhost
192.168.1.5 nefitari.home.lab nefitari
Restart the server after these changes.

Step 4: Install BIND9
If not already installed, install BIND9:


sudo apt-get install bind9
Step 5: Configure BIND9
Configure named.conf.options
Edit the options file to configure DNS IPs:


sudo nano /etc/bind/named.conf.options
Example configuration:


forwarders {
  192.168.1.1;  # Gateway or router
  182.176.39.23;
  182.176.18.13;
  68.87.76.178;
};
Configure named.conf.local
Edit the local configuration file to define forward and reverse zones:


sudo nano /etc/bind/named.conf.local
Example configuration:
allow-query     { localhost;192.168.0.0/16;10.0.0.0/8; }

# Our forward zone
zone "home.lab" {
  type master;
  file "/etc/bind/zones/db.home.lab";
  allow-update { none; };

};

# Our reverse Zone
zone "1.168.192.in-addr.arpa" {
  type master;
  file "/etc/bind/zones/db.192";
  allow-update { none; };

};
Create Zone Files
Create the zones directory and add the zone files:


sudo mkdir /etc/bind/zones
sudo cp /etc/bind/db.local /etc/bind/zones/db.home.lab
sudo nano /etc/bind/zones/db.home.lab
Example zone file content:


$TTL 604800
@        IN         SOA            nefitari.home.lab. webuser.home.lab. (
2 ; Serial
604800 ; Refresh
86400 ; Retry
2419200 ; Expire
604800 ) ; Negative Cache TTL

home.lab.            IN          NS           nefitari.home.lab.
home.lab.            IN          A             192.168.1.5
nefitari                  IN          A            192.168.1.5
gateway                IN          A            192.168.1.1
win7pc                  IN          A            192.168.1.50
www                     IN         CNAME      home.lab.
Configure Reverse Lookup Zone
Copy the reverse lookup file and edit it:


sudo cp /etc/bind/db.127 /etc/bind/zones/db.192
sudo nano /etc/bind/zones/db.192
Example configuration:


$TTL 604800
@                IN             SOA               nefitari.home.lab. webuser.home.lab. (
2 ; Serial
604800 ; Refresh
86400 ; Retry
2419200 ; Expire
604800 ) ; Negative Cache TTL

                 IN               NS            nefitari.
1                IN              PTR           gateway.home.lab.
5                IN              PTR           nefitari.home.lab.
50             IN               PTR            win7pc.home.lab.


Check Zone Files
To check the forward zone:


named-checkzone home.lab /etc/bind/zones/db.home.lab
Output should be:


zone home.lab /IN: loaded serial 2
Ok
To check the reverse zone:


named-checkzone home.lab /etc/bind/zones/db.192
Output should be:


zone home.lab /IN: loaded serial 2
Ok
Configure /etc/resolv.conf
Edit the resolv.conf file to specify the nameserver:


sudo nano /etc/resolv.conf
Example configuration:


nameserver 192.168.1.5
domain home.lab
search home.lab
Update /etc/networking/interfaces
Add the following line to specify the DNS nameserver:


dns-nameservers 192.168.1.5
Restart BIND9
Restart the BIND9 service to apply the changes:


sudo /etc/init.d/bind9 restart
sudo systemctl restart named
sudo systemctl enable named
Check Logs
To ensure there are no errors, check the logs:


tail -f /var/log/syslog
Test Forward Zone
Use host or nslookup to test the forward zone:


host -l home.lab
Output should look like this:


home.lab name server nefitari.home.lab.
home.lab has address 192.168.1.5
gateway.home.lab has address 192.168.1.1
nefitari.home.lab has address 192.168.1.5
win7pc.home.lab has address 192.168.1.50
Test with nslookup:


nslookup home.lab
Output:


Server: 192.168.1.5
Address: 192.168.1.5#53

Name: home.lab
Address: 192.168.1.5
Test with dig:


dig gateway.home.lab
Output should look like this:


;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 35612
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; QUESTION SECTION:
;gateway.home.lab IN A

;; ANSWER SECTION:
gateway.home.lab 604800 IN A 192.168.1.1

;; AUTHORITY SECTION:
home.lab. 604800 IN NS nefitari.home.lab.

;; ADDITIONAL SECTION:
Nefitari.home.lab. 604800 IN A 192.168.1.5

;; Query time: 12 msec
;; SERVER: 192.168.1.5#53(192.168.1.5)
;; WHEN: Thu Aug 8 01:56:25 2013
;; MSG SIZE rcvd: 90
Test Reverse Zone:


host 192.168.1.1
Output should look like this:


1.1.168.192.in-addr.arpa domain name pointer gateway.home.lab
Step 6: Configuring Clients
Windows Clients
Open Network Connections.

Select Change adapter settings.

Right-click on your connection and select Properties.

Select Internet Protocol Version 4 (TCP/IPv4).

Set the following values:

IP address: 192.168.1.50 (or your desired IP)

Subnet Mask: 255.255.255.0

Default Gateway: 192.168.1.1

Preferred DNS server: 192.168.1.5 (your DNS server IP)

Set DNS Suffix to home.lab.

Linux Clients
Edit /etc/network/interfaces:


sudo nano /etc/network/interfaces
Add the following:


auto eth0
iface eth0 inet dhcp
Restart networking:


sudo /etc/init.d/networking restart
If you have a DHCP server, configure it with:


option domain-name "nefitari.home.lab";
option domain-name-server  192.168.1.5;
Step 7: Enable Query Logging in BIND
To enable query logging:


sudo rndc querylog
Check the status:


sudo rndc status
View logs:


tail -f /var/log/syslog
To disable query logging:


sudo rndc querylog
Custom Query Logging
To create custom logs, modify named.conf.local:


sudo nano /etc/bind/named.conf.local
Add:


logging {
    channel mylog_default {
        file "/var/log/mylogs/mylog.log" versions 3 size 12m;
        severity dynamic;
        print-time yes;
    };
    category default { mylog_default; };
};
Create log folder and give permission:


sudo mkdir /var/log/mylogs
sudo chown bind:bind /var/log/mylogs
Create log file or restart the service.

Restart BIND:


sudo /etc/init.d/bind9 restart
Check the custom logs:


tail -f /var/log/mylogs/mylog.log


