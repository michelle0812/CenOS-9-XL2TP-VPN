# CenOS-9-XL2TP-VPN
<code>
This for CentOS Stream 9，其他版本不保證可以用

  第一步：安裝Libreswan與x2ltpd
root# dnf install libreswan xl2tpd -y

  打開vpn需要用到的服務
firewall-cmd --zone=public --add-port=500/udp --permanent ;\
firewall-cmd --zone=public --add-port=1701/udp --permanent ;\
firewall-cmd --zone=public --add-port=4500/udp --permanent ;\
firewall-cmd --zone=public --add-masquerade --permanent ;\
systemctl restart firewalld

  備份既有的vpn相關設定檔
cp /etc/ipsec.conf /etc/ipsec.conf.bsd ;\
cp /etc/ipsec.secrets /etc/ipsec.secrets.bsd ;\
cp /etc/sysctl.conf /etc/sysctl.conf.bsd ;\
cp /etc/xl2tpd/xl2tpd.conf /etc/xl2tpd/xl2tpd.conf.bsd ;\
cp /etc/ppp/chap-secrets /etc/ppp/chap-secrets.bsd

第二步：編輯ipsec與xl2tp相關設定檔
root# vi /etc/ipsec.conf

version 2.0

config setup
  virtual-private=%v4:192.168.0.0/16
  ikev1-policy=accept
  uniqueids=no

conn shared
  left=%defaultroute
  leftid=$public_ip
  right=%any
  encapsulation=yes
  authby=secret
  pfs=no
  rekey=no
  keyingtries=5
  dpddelay=30
  dpdtimeout=120
  dpdaction=clear
  ikev2=never
  ikelifetime=24h
  salifetime=24h
  sha2-truncbug=no

conn L2TP-PSK
  auto=add
  leftprotoport=17/1701
  rightprotoport=17/%any
  type=transport
  also=shared

include /etc/ipsec.d/*.conf
</code>
