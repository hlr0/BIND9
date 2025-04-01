# BIND9 
## UBUNTU: Complete DNS Server Configuration with hostname

```
sudo apt update -y
sudo systemctl stop ufw && systemctl disable ufw
sudo firewall-cmd --permanent --add-service=dns
sudo firewall-cmd --permanent --add-port=53/udp
sudo firewall-cmd --reload
sudo dnf install bind9 bind9-utils -y
sudo systemctl start named
sudo systemctl enable named
sudo systemctl status named
```

First of all change the ip address of your server form DHCP to STATIC 
```
sudo nano /etc/network/interfaces
```
for this use the following command
```
auto eth0
iface eth0 inet static
address 192.168.1.5
netmask 255.255.255.0
network 192.168.1.0
broadcast 192.168.1.255
gateway 192.168.1.1
# dns-nameservers
*I am leaving dns-nameserver empty and is commented we will come to it later.
```

Restart networking daemons

```
Sudo /etc/init.d/networking restart
```

Before configuring a DNS server in linux Ubuntu you have to make domain name first and then you will proceed. First you will check your hostname command for this is

```
Sudo nano /etc/hostname
```

My Server has the following name

```
nefitari
```
(This is my Ubuntu server hostname yours might be different .You can change this according to your need)

Now after hostname, you have to make domain name for your server. Say servername.domain.com it is better practice that whenever you are configuring server for home use or so, do not use .com but .hom or .net or .lab or whatever you like. Give the below command

```
Sudo nano /etc/hosts
```
```
127.0.0.1 localhost
192.168.1.5 nefitari.home.lab nefitari
```

In my file 127.0.0.1 is for localhost and I have changed the second IP address 127.0.1.1 with my server IP that is 192.168.1.5 now I enter my domain name having my hostname nefitari first then my domain name home.lab and then alias nefitari. You can select of your own, hostname.abc.net or hostname.home.lan etc. but remember changing to this file need to restart your server and then login. Restart is must
\
After installation just configure the below files step by step\
\
Named.conf.options\
Named.conf.local\
Named.conf.resolv.conf\
\
Now configure file named.conf.options This file is use for DNS IPs It mean that your server must connect to some DNS outside. When you buy domain name from ISP’s they normally gives you their own DNS IPs. You can use open DNS IPs of google or so. In my case I am using my own ISP DNS IPs.
```
Sudo nano /etc/bind/named.conf.options
```
```
forwarders {
# Give here your ISP DNS IP’s
192.168.1.1; # gateway or router
182.176.39.23;
182.176.18.13;
68.87.76.178;
};
```
Save the file and exit using control x press y and overwrite the file\
\
Now edit the file named.conf.local This is the file in which we define forward zones and reverse zones. It means that when we enter domain name it will translate it into IP address and when we enter IP address it will simply convert it into name.

```
Sudo nano /etc/bind/named.conf.local
```
```
#Our forward zone
zone "home.lab" {
type master;
file "/etc/bind/zones/db.home.lab";
};

#Our reverse Zone
#Server IP 192.168.1.5
zone "1.168.192.in-addr.arpa" {
type master;
file "/etc/bind/zones/db.192";
};
```

Save the file and exit\
Now we will make these two database files db.home.lab and db.192 in zones folder\
First make the directory zones in /etc/bind/

```
Sudo mkdir -p /etc/bind/zones
```

Before making files let me be clear to you that I have different devices

```
Devices IPs
Server itself 192.168.1.5
Gateway 192.168.1.1
Win7pc 192.168.1.50
```

Now in zones directory we will create two files first db.home.lab. I am just copying the db.local already present in /etc/bind folder to zones folder by changing its name to db.home.lab. I will put these IP’s in my db.home.lab file. Let’s start

```
Sudo cp /etc/bind/db.local /etc/bind/zones/db.home.lab
```

Now use the command below to edit the file

```
Sudo nano /etc/bind/zones/db.home.lab
```

```
;
; BIND data file for local loopback interface
;
$TTL 604800
@        IN         SOA            nefitari.home.lab. webuser.home.lab. (
2 ; Serial
604800 ; Refresh
86400 ; Retry
2419200 ; Expire
604800 ) ; Negative Cache TTL
;
home.lab.            IN          NS           nefitari.home.lab.
home.lab.            IN          A             192.168.1.5
;@                       IN          A             127.0.0.1
;@                       IN          AAAA        ::1
nefitari                  IN          A            192.168.1.5
gateway                IN          A            192.168.1.1
win7pc                  IN          A            192.168.1.50
www                     IN         CNAME      home.lab.
Save it and exit
```
check the configuration file 

```
named-checkconf /etc/named.conf.local
```
Webuser.home.lab. is the email who will access name server. You can write any name instead webuser like admin, root or host master etc.
home.lab. is my NS means name server
home.lab.changing to IP 192.168.1.5\
\
@ IN A 127.0.0.1 and AAAA ::1 can be comment out you should not need it because db.local is already present in /etc/bind it is just a copy of that file. So no need you can delete it\
\
Changing Nefitari to IP 192.168.1.5
\
Gateway to IP 192.168.1.1

Win7pc you can name your windows PCs or Linux Clients to any name but remember IP of that client must correctly be inserted into file. In my case I gave IP of windows PC 192.168.1.50\
\
Last, I am using CNAME means canonical name it is just an alias to nefitari. Means that you can access your server by entering www.home.lab instead nefitari.home.lab . You can omit this or comment it. It is just up to you.
\
Now create reverse lookup zone file
```
Sudo cp /etc/bind/db.127 /etc/bind/zones/db.192
```
Now use the command below to edit the file

```
Sudo nano /etc/bind/zones/db.192
```
```
;
; BIND reverse data file for local loopback interface
;
$TTL 604800
@                IN             SOA               nefitari.home.lab. webuser.home.lab. (
2 ; Serial
604800 ; Refresh
86400 ; Retry
2419200 ; Expire
604800 ) ; Negative Cache TTL
;
                 IN               NS            nefitari.
1                IN              PTR           gateway.home.lab.
5                IN              PTR           nefitari.home.lab.
50             IN               PTR            win7pc.home.lab.
```
Save it and exit
\
Now when you are done with your zone file you have to check it whether it is working correctly or not by entering the command below for forward zone file
\
```
named-checkzone home.lab /etc/bind/zones/db.home.lab
```
```
zone home.lab /IN: loaded serial 2
Ok
```
Now check the reverse zone file
```
named-checkzone home.lab /etc/bind/zones/db.192
```
```
zone home.lab /IN: loaded serial 2
Ok
```
If the output of your named-checkzone is same as above then it is working fine otherwise you made some mistake in file.
\
Now edit the file resolv.conf
```
Sudo nano /etc/resolv.conf
```
```
Nameserver 192.168.1.5
domain home.lab
search home.lab
```
Enter the following lines into to your resolv.conf file and save it\
\
Now come to dns-nameservers (/etc/networking/interfaces)\
you will now add the following code to /etc/networking/interfaces
```
dns-nameservers 192.168.1.5
```
reason for this is that whenever you restart server /etc/resolv.conf file wash its contents
Restart the bind

```
sudo /etc/init.d/bind9 restart
sudo systemctl restart named
sudo systemctl enable named
```
After bind start check your setting in log file

```
tail -f /var/log/syslog
```
it must not have any error in the log
Checking forward zones
```
host –l home.lab
```
Output should like this
```
home.lab name server nefitari.home.lab.
home.lab has address 192.168.1.5
gateway.home.lab has address 192.168.1.1
nefitari.home.lab has address 192.168.1.5
win7pc.home.lab has address 192.168.1.50
```
Now use NSLOOKUP
```
nslookup home.lab
```
```
Server: 192.168.1.5
Address: 192.168.1.5#53

Name: home.lab
Address: 192.168.1.5
Use DIG

```
```
Dig gateway.home.lab
```
```
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
```
Output should similar to the above, check status: NOERROR means it is resolving check ANSWER SECTION: gateway.home.lab is resolved into 192.168.1.1
Checking reverse zone

```
host 192.168.1.1
```

```
1.1.168.192.in-addr.arpa domain name pointer gateway.home.lab
```
If it gives you an error like below

```
host 1.1.168.192.in-addr.arpa. not found: 3(NXDOMAIN)
```
This means that you made some mistake in /etc/bind/named.conf.local file in reverse zone If your server IP is 192.168.1.5 then your reverse zone looks like this
```
zone "1.168.192.in-addr.arpa" {
correct ip reversing
};
```
Sometime people made mistake in reversing the ip like (just an example)
```
zone "0.168.192.in-addr.arpa" {
incorrect ip reversing
};
```
Use NSLOOKUP

```
nslookup 192.168.1.1
```
```
Server: 192.168.1.5
Address: 192.168.1.5#53

1.1.168.192.in-addr.arpa name=gateway.home.lab
```
If you get NXDOMAIN or SERVFAIL like errors it means that one of your zone file is not working correctly\
\
Test Your Server to Outside World\
Now you can ping ubuntu.com or dig ubuntu.com for the first time it will take several miliseconds to resolve the name ubuntu.com but when you run it second time, it will take form 1 to 10 mili seconds, its normal time and it means that your DNS is working properly\

#Configuring clients
##windows side

open network connections\
select change adapter settings\
select properties\
select internet protocol version IPv4\
\
and here give the IP address (in my case it is 192.168.1.50 have you remember win7pc)\
\
IP address 192.168.1.50\
Subnet Mask 255.255.255.0\
Default Gateway 192.168.1.1\
primary DNS 192.168.1.5 (my new BIND DNS server ip)\
select Advance (in the same window)\
select DNS tab\
Type in the text box below here In DNS Suffix for this connection:home.lab\
click ok\
click on validate setting upon exit\
click ok\
\
and you are done with it open CMD\
```
ping gateway
```
it must gives you some replies similarly

```
ping 192.168.1.1 or 5
```
it must gives you some replies\
you can use NSLOOKUP\
```
nslooup gateway
```

# LINUX CLIENTS

Code:
sudo nano /etc/network/interfaces
type the following lines
Code:
auto eth0
iface eth0 inet dhcp
Now restart Network Deamons
Code:
Sudo /etc/init.d/networking restart
to force client renew IP command
Code:
sudo dhclient -r
Now obtain fresh IP:
Code:
sudo dhclient
If you are running DHCP server on your system then enter the domain name and name server in dhcpd.conf file for example I have DNS server named nefitari.home.lab and IP address is 192.168.1.5 like as under

Code:
option domain-name “nefitari.home.lab”;
option domain-name-server  192.168.1.5;
How to enable query logging in BIND
To enable query logging execute rndc querylog command:
Code:
Sudo rndc querylog
Check out query logging status by executing command:
Code:
Sudo rndc status
Now you can view queries:
Code:
tail -f /var/log/syslog
To disable it execute command again.
Code:
Sudo rndc querylog
Our own Query Logging in BIND

Code:
Sudo nano /etc/bind/named.conf.local
Add the following lines at the bottom of the file. You can use any channel to produce log file. You can use more than one channel as well.
Code:
logging {
    channel mylog_default {
        file "/var/log/mylogs/mylog.log" versions 3 size 12m;
        severity dynamic;
        print-time yes;
    };
category default { mylog_default; };
};
After saving the file go to /var/log/ and make a mylogs folder and give it bind permission so that bind can write to it.
Code:
Sudo mkdir /var/log/mylogs
Code:
sudo chown bind:bind /var/log/mylogs
Now after this you can make file mylog.log by using sudo nano mylog.log and save it in the directory mylogs or when you edit Apparmor it create the file automatically after restarting the bind, But at this time when you restart the bind it will not start and show fail. Because before named daemon can write to the new log file the AppArmor profile must be updated. First, edit
Code:
Sudo nano /etc/apparmor.d/usr.sbin.named
And add:
Code:
/var/log/mylogs/mylog.log w,
Next, reload the profile:
Code:
sudo cat /etc/apparmor.d/usr.sbin.named | sudo apparmor_parser -r
It may give you some warnings
Now restart bind
Code:
Sudo /etc/init.d/bind9 restart
To check logs
Code:
Tail –f /var/log/mylogs/mylog.log
