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
```
sudo mount.nfs -o port=19815,mountport=19815,vers=3,sync,tcp 192.168.3.1:/data /mnt/nfs
sudo mount.cifs -o username=user1,uid=1000,gid=1000 //192.168.3.1/data /mnt/smb
sudo mount.davfs -o username=user1,uid=1000,gid=1000 http://192.168.3.1 /mnt/webdav
```
### IPSec-Server
```
wget https://git.io/vpnsetup-centos -O vpn.sh && sh vpn.sh
默认为ikev1证书，若配置ikv2参考如下:
ikev2.sh --auto
IKEv2 guide:       https://git.io/ikev2
若自定义IPsec预共享密钥，vpn用户名和密码，参考如下:
wget https://git.io/vpnsetup-centos -O vpn.sh
VPN_IPSEC_PSK='IPsec预共享密钥' \
VPN_USER='VPN用户名' \
VPN_PASSWORD='VPN密码' \
sh vpn.sh
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
redial = yes
redial timeout = 10
autodial = yes
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
nodefaultroute
usepeerdns
connect-delay 5000
name "VPN用户名"
password "VPN密码"
EOF
chmod 600 /etc/ppp/options.l2tpd.client

在 /etc/strongswan/strongswan.d/charon.conf文件的start-scripts内增加：
    start-scripts {
        init_ecloud = /usr/sbin/strongswan up ecloud
    }

在/usr/lib/systemd/system/xl2tpd.service文件的[Service]部分增加：
ExecStartPre=/usr/bin/mkdir -p /var/run/xl2tpd

systemctl daemon-reload
systemctl restart strongswan
systemctl restart xl2tpd

systemctl enable strongswan
systemctl enable xl2tpd

在/etc/ppp/ip-up文件末尾增加移动云vpc子网段(.e.g:192.168.0.0/24)路由规则:
命令行:
sed -i '/exit 0/i\ip route add 192.168.0.0/24 dev $REALDEVICE' /etc/ppp/ip-up
cat /etc/ppp/ip-up
...
[ -x /etc/ppp/ip-up.local ] && /etc/ppp/ip-up.local "$@"

ip route add 192.168.0.0/24 dev $REALDEVICE
exit 0

重启服务
systemctl enable xl2tpd
```
