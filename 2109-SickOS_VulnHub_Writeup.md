# SickOS 1.1 Walkthrough (Getting root)
-----------------------------------------
- OS (Box): Ubuntu (SickOS 1.1 - https://www.vulnhub.com/entry/sickos-11,132/)
- Objective: Obtain root user, and find flag in /root/ directory (without using any walkthroughs).


# 1. Nmap
- **We could not find the IP address for the box using ‘nmap -sn’ or ‘-Pn’ scan. But using sudo it is possible to scan for the box.**
```
(kali@kali)-[~] $ nmap -sn 192.168.56.0/24 
Starting Nmap 7.91 ( https://nmap.org ) at 2022-02-19 15:14 ESTIU 
Nmap scan report for (192.168.56.1) Host is up (0.00024s latency). 
Nmap scan report for 192.168.56.3 cm Host is up (0.00013s latency). Nmap done: 256 IP addresses (2 hosts up) scanned in 2.97 seconds
```
  > 192.168.56.11 is identified as SickOS. (.1 is the host machine, .2 is the VirtualBox router (https://router-network.com/ip/192-168-56-2), and .3 is Kali.)

- **Using ‘nmap -sV’ we can see the running services. Squid proxy on Port 3128 seems promising, so let's look into that.**

```
(kali@kali) - [~] $ sudo nmap -sV 192.168.56.11 
Starting Nmap 7.91 ( https://nmap.org) at 2022-02-19 15:51 EST 
Nmap scan report for 192.168.56.11 Host is up (0.00031s latency) 
Not shown: 997 filtered ports PORT STATE SERVICE VERSION 
22/tcp open ssh OpenSSH 5.9p1 Debian Subuntul.1 (Ubuntu Linux; protocol 2.0) 
3128/tcp open http proxy Squid http proxy 3.1.19 
8080/tcp closed http-proxy MAC Address: 08:00:27:AB:F1:24 (Oracle VirtualBox virtual NIC) Service Info: 05: Linux; CPE: cpe:/o:linux:linux kernel
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ . Nmap done: 1 IP address (1 host up) scanned in 15.92 seconds
```
-----------------------------------------
# 2. Proxy
- FoxyProxy is a Firefox extension which automatically switches an internet connection across one or more proxy servers based on URL patterns; put simply, FoxyProxy automates the manual process of editing Firefox's Connection Settings dialog.
Let’s use the extension to connect to the Squid http proxy by entering in the box’s IP address and Port 3128. We are able to connect to the proxy server without any type of authentication. Then we’re able to access the running web service, which is just a blank page with text.


![img](https://i.ibb.co/xMzxv0t/Screenshot-2.png)

![img](https://i.ibb.co/dM4qF3n/Screenshot-1.png)

-----------------------------------------

# 3. Dirb
- The tool ‘dirb’ (with -p specified) lists a few 200 codes for you to explore. The page robots.txt shows a disallow on /wolfcms/ therefore let’s look further into the box’s CMS. 
