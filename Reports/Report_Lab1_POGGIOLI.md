# Report Labs / Flavio POGGIOLI

## Table of contents

  - [Report Labs / Flavio POGGIOLI](#report-labs--flavio-poggioli)
  - [Table of contents](#table-of-contents)


Goal : find 8 vulnerabilities flags beginning by "FLAG{".

## Find IP of the target
  
  ```bash
  $ sudo netdiscover -r 10.0.2.0/24
  ```
![picture 3](../images/d3d871b56ca16ee5b9b9ebaad68ac00140d7bc4045115ce2f262985e4eac8bfb.png)  

Our target is 10.0.2.4. 

The 2 first is VBox address (network and dhcp).

## List all open ports
  
  ```bash
  $ sudo nmap -p- 10.0.2.4
  ```
  ![picture 4](../images/46b46c45bff383ec5c6f5ed005114fd16f8d8b3d5ce19dd1c613166813623a50.png)  


```bash
$ sudo nmap -sV -p 21,22,80,9090,13337,22222,60000 10.0.2.4
```

That command operates a scan on the specified ports on a host or network and attempt to determine the version of the active services on those ports. 

Ports scanned : 21,22,80,9090,13337,22222,60000

![picture 5](../images/d1d862eef18773ac04c81d1ca694d994baad9c448add4eb7b2b20c4df6e615e3.png)  

We can find the version of some services, and for services which return any version, we can see the data of the service.

## Flag 1 - FLAG{THERE IS NO ZEUS, IN YOUR FACE!}

With the different ports scanned on previous step, we can try to connect to the services with different manners (ssh, telnet, http)...

I tried things with the port 80, which print an image of Morty, but nothing.

![picture 8](../images/0e72ea710bd116daa472f9cbab190a5071a7dd6aa96788c55448eddc8d2f1d1f.png)  

The port 9090 is a web service, and we found a flag : FLAG{THERE IS NO ZEUS, IN YOUR FACE!}.

![picture 6](../images/2e0080edc3cada760aadc48c3b7478ccf34f32abb33a57421f539f132b92138a.png)  

## Flag 2 - FLAG:{TheyFoundMyBackDoorMorty}

With the browser, on port 13337, we can find a flag : FLAG:{TheyFoundMyBackDoorMorty}.

![picture 9](../images/c104793dc2e75a413a996b150efa6903a318b34cd8b36d8a9ae7b7fe4ac1cf5c.png)  

Nothing on other ports with the browser.

## Flag 3 - FLAG{Yeah d- just don't do it.}

For the port 80, which host a web server, we can dig to find a something in arborescence directory.

```bash
$ dirb http://10.0.2.4
```

![picture 10](../images/59b6932517ac65bc914e74865ca3a0733f9244c335bfd5d3cab2fed56ef1c743.png)  

We find files and a directory : http://10.0.2.4/passwords, this directory is listable.
In this directory, we can find a file named FLAG.txt which contains : FLAG{Yeah d- just don't do it.}

![picture 12](../images/10cbe76c0d34fcde81ae7f9153943de609c58d8d0be631595babcc9ead99598a.png) 

![picture 14](../images/e779f9cb8b87ebded0217381553daa5eb022e2b370be5584cec4576514d14e2c.png)


## Flag 4 - FLAG{Flip the pickle Morty!}

On the ports scanned, the data returned from 60000 port shows a reverse shell with his fingerprint. We can't do anything with it on browser, ssh, telnet, ftp... So we can use netcat to connect to the port and try to find something.

```bash
$ nc 10.0.2.4 60000
```
Through that, we can interact with a shell, and we can find a flag : FLAG{Flip the pickle Morty!}.

![picture 11](../images/6b2f48934cc9126698ec9fa41c1378ecc38d360d856960742912407b2dd3bdfa.png)  

## Flag 5 - FLAG{Whoa this is unexpected} 

For the port 21, a more precise analysis with nmap can show us that the ftp service is vulnerable to anonymous login.

![picture 15](../images/985bccabaf0ada6390eeb8b9d2f3204a90b14587af9de83b99e45365295c5cbe.png)  

We can see that we can connect as anonymous on ftp, 2 files are present, FLAG.txt and pub.

![picture 16](../images/d12a4b1da91e399d496348f2c8fd588de6a5fe9275f722c7049fc12c1966ad49.png)  

I download FLAG.txt because we don't have any permission on ftp server to open it.

![picture 17](../images/ef129d29883cbd50bfaa07252370c03cc3196233b606a567799433f12b50d49d.png)  

And we can discover a flag : FLAG{Whoa this is unexpected}.

## Flag 6 - FLAG{Get off the high road Summer!}

In password directory, there is also a file named "passwords.html". After an analysis, we can see in console that a password is hidden in the code.

![picture 13](../images/8ca44e2a59a8f86bc80cec643bf16ebaac1ebc22135ba5d4c643cc5f6a008fa7.png) 


On http://10.0.2.4/cgi-bin/tracertool.cgi found on step Flag3, there is a prompt which allows to do tracert. But, using the command on the prompt, we can interrupt the command and execute malicious command with ";".

![picture 18](../images/797927299f718d85e9ac05c1e0afdfeea16067f5cfe925f045367316095e8581.png)  

So, with the password "winter", we can maybe connect with ssh to the host, so we must find username on /etc/passwd.

But when the command "; ls /etc/passwd" is executed, we can see a real cat.

![picture 19](../images/6a1704eb179441519ff6324c9bafb6799645af7ded3734f6274208c6f2c216d0.png)  

To get around this, we can use the command "; less /etc/passwd" to find the username.

![picture 20](../images/7354f46bdd59bfcbea0f13ce50154fdfddbc4287aa26000521713077ba475f87.png)  


The connection ssh is closed on port 22, but we can try to connect on port 22222.

![picture 21](../images/e0590b6e267d81a81390dccecd6f8594ec73538e4840abbdcc90f07ebed56863.png)  

The user is Summer (opposite to the password "winter") and we can connect to it.

![picture 22](../images/2346ffaadc886004bb4d633ba3c26182a1110595fb33d3717cb15dc9db9497c7.png)  

The command cat is alse printing a real cat so we can use the command "; less /home/Summer/FLAG.txt" to find a flag : FLAG{Get off the high road Summer!}.

## Flag 7 - FLAG: {131333} 

With a SSH access, we can access to Morty home directory.

![picture 23](../images/496ce5fb2f7a68967ed299972b4b073cc1c25a4ee2da47d03ded61162c6010c0.png)  

There is a password needed to unzip journal.txt.zip.

![picture 24](../images/4d15f2ee5e459d7a0c44a0927616c16a937220e8a18798501cc4a689e26d1d21.png)  

The password is in Safe_Password.jpg file.

![picture 26](../images/c0c727d514ca420c74dfa39c84999ab88a0a9cb043be3bea8b3ffe3d75719b42.png)  

Meeseek is the password and we can find a flag : FLAG: {131333} 

![picture 27](../images/0a6acef3189d0cd8c1bd75c662ddb184e71e2ff350a86739658d3eb1716da2c2.png)  


### Flag 8 - FLAG{And Awwaaaaayyyyy we Go!}

![picture 28](../images/55f3005acd5977ab189e09527331fa05d40bdad9b4067d7710f88ea78ae4fd74.png)  

There is an executable on personal folder of Rick, which contains a directory named "RicksSAFE".

A key is needed to open the safe, and we can find to the older flag : 131333.

But, the file is executable with the libary libmcrypt4.

![picture 29](../images/658477b4e15b978da7d89745627dc71b7df91f22716e670b6aee5af0284c565d.png)  

## Flag 9 - FLAG secret 
I was also given a password hint for Rick’s password. A quick Google search found that the band name was “The Flesh Curtains”.

So, I create a python script to create the possible password.

```python
from string import ascii_uppercase
for c in ascii_uppercase:
    for x in range(0, 10):
        print str(c) + str(x) + "Flesh"
        print str(c) + str(x) + "Curtains"
```

I redirect the output to a file :
```bash
python3 scriptpython.py > password.txt
```

![picture 30](../images/4c22d24e81edf2411389a7c267b7b7b7f02fb72295ed071e0a955b2515078960.png)  


We can now try with Hydra to bruteforce the password :
```bash
hydra -s 22222 -v -V -l RickSanchez -P password.txt -t 16 10.0.2.4 ssh
```


![picture 33](../images/f2043518aa192a3ea89ff8511234ecc786ba5d8bfb14f56577cbf14bd7dcc6af.png)  

And we can find finally a file named FLAG.txt in the directory /root, where RickSanchez has the permissions.

![picture 32](../images/7f044d25acba6334df9d8f24b5acbb06a7bb61588a8aa1964c70644a2ddece79.png)  
