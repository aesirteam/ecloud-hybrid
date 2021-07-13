# ecloud-hybrid
### OSS
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
### IPSec-Server
```
```
### IPSec-Client
yum install -y epel-release  
yum --enablerepo=epel install -y strongswan xl2tpd net-tools  
```
cat > /etc/strongswan/ipsec.conf <<EOF
conn ecloud
  auto=start
  keyexchange=ikev1
  authby=secret
  type=transport
  left=%defaultroute
  leftprotoport=17/1701
  rightprotoport=17/1701
  right="VPN服务器IP"
  ike=aes128-sha1-modp2048
  esp=aes128-sha1
EOF

cat > /etc/strongswan/ipsec.secrets <<EOF
: PSK "IPsec预共享密钥"
EOF
chmod 600 /etc/strongswan/ipsec.secrets

cat > /etc/xl2tpd/xl2tpd.conf <<EOF
[lac ecloud]
lns = "VPN服务器IP"
ppp debug = no
pppoptfile = /etc/ppp/options.l2tpd.client
length bit = yes
EOF

cat > /etc/ppp/options.l2tpd.client <<EOF
ipcp-accept-local
ipcp-accept-remote
refuse-eap
require-chap
noccp
noauth
mtu 1280
mru 1280
noipdefault
#defaultroute
usepeerdns
connect-delay 5000
name "VPN用户名"
password "VPN密码"
EOF
chmod 600 /etc/ppp/options.l2tpd.client

在/usr/lib/systemd/system/xl2tpd.service文件的[Service]部分增加：
ExecStartPre=/usr/bin/mkdir -p /var/run/xl2tpd

systemctl daemon-reload
systemctl restart strongswan
systemctl restart xl2tpd
```
