# CenOS-9-XL2TP-VPN 筆記
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
  # ---- 設定檔開始與結束
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
  # ---- 設定檔開始與結束
  
  root# vi /etc/ipsec.secrets
  # ---- 設定檔開始與結束
include /etc/ipsec.d/*.secrets
%any: PSK "ThisIsTheSharingKey"
  # ---- 設定檔開始與結束

  
  root vi /etc/sysctl.conf #加入以下，並替換成自己的網卡代號
  # ---- 設定檔開始與結束
net.ipv4.conf.ens160.rp_filter = 0
net.ipv4.conf.ens224.rp_filter = 0
net.ipv4.conf.ppp0.rp_filter = 0
net.ipv4.conf.ip_vti0.rp_filter = 0
net.ipv4.conf.lo.rp_filter = 0
  # ---- 設定檔開始與結束
  
  root# vi /etc/xl2tpd/xl2tpd.conf
  # ---- 設定檔開始與結束
[global]
ipsec saref = yes
[lns default]
ip range = 192.168.253.211-192.168.253.230
require chap = yes
refuse pap = yes
require authentication = yes
name = LinuxVPNserver
ppp debug = yes
pppoptfile = /etc/ppp/options.xl2tpd
length bit = yes
  # ---- 設定檔開始與結束
  root# vi /etc/ppp/chap-secrets
  # ---- 設定檔開始與結束
# Secrets for authentication using CHAP
# client        server  secret                  IP addresses
test-account    *       account-password        *
  
</code>
