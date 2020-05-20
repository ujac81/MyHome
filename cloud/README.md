# CLOUD
Configuration notes about nextcloud

## Setup:
* Nextcloud hosted on raspberry PI 4 running raspbian buster.
* Lighttpd HTTP server
* MariaDB database server
* SSD connected via USB
* Fritzbox setup for port forwarding of HTTP and HTTPS (ports 80 and 443) with fixed IP address for cloud.

## Installation
```bash
apt update
apt upgrade
apt install certbot mariadb-server php7.3 lighttpd
```

## Setup SSD
Partition SSD and format with ext4 file system.
Add to fstab using uuid found via
```bash
ls -la /dev/disk/by-uuid/
```
Add to /etc/fstab:
```bash
UUID="03d7af34-d5d9-48d1-8ca2-4ab57b165e30" /media/cloud-ssd    ext4    defaults,nofail 0 0
```
and create mount point
```bash
mkdir -pv /media/cloud-ssd
```

## Setup MariaDB
Move /var/lib/mysql to SSD and link it back
```bash
systemctl stop mariadb.service
mv /var/lib/mysql /media/cloud-ssd
ln -s /media/cloud-ssd/mysql /var/lib/mysql
systemctl start mariadb.service
```

## Setup Nextcloud
Download latest stable version: https://nextcloud.com/install/#instructions-server
And unzip it to /media/cloud-ssd/www

## Setup Certbot
Prepare certbot like found on google and change cron job to merge pem files:
/etc/cron.d/certbot
```
0 */12 * * * root test -x /usr/bin/certbot -a \! -d /run/systemd/system && perl -e 'sleep int(rand(43200))' && certbot -q renew && cat /etc/letsencrypt/live/MY_DOMAIN/privkey.pem /etc/letsencrypt/live/MY_DOMAIN/cert.pem > /etc/letsencrypt/live/MY_DOMAIN/merged.pem
```
## Setup lighttpd
Use setup, configrued by pihole before.
Add /etc/lighttpd/conf-available/10-ssl_custom.conf
```
# /usr/share/doc/lighttpd/ssl.txt

server.modules += ( "mod_openssl" )

$SERVER["socket"] == "0.0.0.0:443" {
        ssl.engine  = "enable"
        ssl.pemfile = "/etc/letsencrypt/live/MY_DOMAIN/merged.pem"
        ssl.ca-file = "/etc/letsencrypt/live/MY_DOMAIN/chain.pem"
        server.name = "MY_DOMAIN"
        ssl.cipher-list = "HIGH"
        server.document-root = "/media/cloud-ssd/www/nextcloud"
        errorlog.filename = "/var/log/lighttpd/nextcloud_error.log"
        accesslog.filename = "/var/log/lighttpd/nextcloud_acces.log"
}
```