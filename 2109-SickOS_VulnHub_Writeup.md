# SickOS 1.1 Walkthrough (Getting root)
-----------------------------------------
- OS (Box): Ubuntu (SickOS 1.1 - https://www.vulnhub.com/entry/sickos-11,132/)
- Objective: Obtain root user, and find flag in /root/ directory (without using any walkthroughs).


# 1. Nmap
### We could not find the IP address for the box using ‘nmap -sn’ or ‘-Pn’ scan. But using sudo it is possible to scan for the box.
   ```
   (kali@kali)-[~] $ nmap -sn 192.168.56.0/24 Starting Nmap 7.91 
   ( https://nmap.org ) at 2022-02-19 15:14 ESTIU Nmap scan report for (192.168.56.1) Host is up (0.00024s latency). Nmap scan report for 192.168.56.3 cm Host is up (0.00013s        latency). Nmap done: 256 IP addresses (2 hosts up) scanned in 2.97 seconds
  ```
  > 192.168.56.11 is identified as SickOS. (.1 is the host machine, .2 is the VirtualBox router (https://router-network.com/ip/192-168-56-2), and .3 is Kali.)

