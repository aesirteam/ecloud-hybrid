# ecloud-hybrid
yum install -y bridge-utils xdg-utils net-tools  httpd-tools unzip
```
touch /etc/htpasswd
htpasswd -B /etc/htpasswd admin
```

```
cd /etc/ssl/certs
openssl genrsa -out rclone.key 4096
openssl req -new -key rclone.key -out rclone.csr
openssl x509 -req -days 3650 -in rclone.csr -signkey rclone.key -out rclone.crt
```

```
yum install -y samba

groupadd sambashare
chgrp sambashare /data
useradd -M -d /data -s /usr/sbin/nologin -G sambashare user1
smbpasswd -a user1
smbpasswd -e user1
```
