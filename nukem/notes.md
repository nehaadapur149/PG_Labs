# Nukem

* nmap

![a](./images/a.png)

* wordpress site . wpscan reveals vulnerable plugin

![b](./images/b.png)

* exp -> https://www.exploit-db.com/exploits/48979

![c](./images/c.png)

![d](./images/d.png)

* linpeas.sh -> reveals db creds

```
define( 'DB_USER', 'commander' );
define( 'DB_PASSWORD', 'CommanderKeenVorticons1990' );
```
* There is also suid set for dosbox -> https://gtfobins.github.io/gtfobins/dosbox/
* We can overwrite /etc/passwd or /etc/sudoers 

* There is internal VNC service running on port 5901

![e](./images/e.png)

* We use ssh rempte port forwarding to access VNC service from our machine.

![f](./images/f.png)

* Used db_password.

![g](./images/g.png)

![h](./images/h.png)

* Overwrite /etc/sudoers file by allowing root access to everyone.

```
mount c /
c:
```

![i](./images/i.png)

* sudo su to get root access

![j](./images/j.png)
 
