# SickOS 1.1 Walkthrough (Getting root)
-----------------------------------------
- OS (Box): Ubuntu (SickOS 1.1 - https://www.vulnhub.com/entry/sickos-11,132/)
- Objective: Obtain root user, and find flag in /root/ directory (without using any walkthroughs).


# 1. Nmap
We Could not find the IP address for the box using ‘nmap -sn’ or ‘-Pn’ scan. But using sudo it is possible to scan for the box.
`(kali@kali)-[~] $ nmap -sn 192.168.56.0/24`
