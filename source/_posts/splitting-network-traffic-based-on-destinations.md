---
title: 野生思科分离隧道
url: 817.html
id: 817
categories:
  - IT
date: 2018-10-06 11:40:47
tags:
---

**写在前面的话：每一个员工都应完全遵守所在组织的IT安全策略。本文只为探讨一种可能性，请不要看完了以后就去放飞自我，作死可能会被开除，造成损失的可能还会被索赔甚至判刑。作者不负任何责任，以上不同意的请立刻按Ctrl+W（Command+Ws）.**

<!-- more -->

~~要作死的话就低调点，别写个文章放网上。~~
很多组织（企业、学校、政府等）都会要求员工在工作的时候（尤其是小型办公室/家庭办公SOHO）接入虚拟专用网（VPN），用尽可能低的成本尽量使得数据在经过公有的互联网时得到保护。其核心技术之一，隧道（tunneling），在VPN客户端与接入端之间建立加密的连接，将数据重新封装之后发送。 有的组织在接入VPN用户时使用了`tunnelall`的策略，这时，VPN用户的所有流量（访问组织内网、访问互联网）都将使用隧道，VPN用户的本地网络、本地ISP将不可使用，也称`full-tunnel`. 也有组织使用`隧道分离(split-tunnel)`的策略，只有组织内网的流量使用隧道，VPN用户的本地网络、本地ISP仍然可用。 随着互联网上的安全威胁日益发展，以及组织对于合规、安全审计等方面的考虑，越来越多的组织使用`full-tunnel`来接入VPN用户，思科的ASA也将此策略设置为默认。 
道理我都懂，但是我的NAS用不了怎么办？我玩扫雷联网速度慢怎么办？我喜欢钻研科技网站po\*nhub怎么办？本文提供了一种思路，通过使用Linux搭建野生VPN服务的办法，将组织的`full-tunnel`分割，去往组织内网的流量与VPN用户本地网络分割开。
![Y*NdA lauunL IltÄA ](https://cdn.beijing2b.com/wp-content/uploads/2018/10/ynda-lauunl-iltaa.png) ![野 生 VPN 接 入 也 是 组 织 VPN 客 尸 端 野 生 Split Tunnel 示 意 图 组 织 VPN 接 入 加 隧 道 ](https://cdn.beijing2b.com/wp-content/uploads/2018/10/vpn-vpn-split-tunnel.png)

### 环境介绍

所需软件都是免费的。硬件需要一个Linux机器，闲置的古老笔记本什么的都可以。我正好有一台闲置的斐讯T1电视盒子，去年占小便宜拿回来的，还木有使用过。这个盒子安装Linux还是蛮好的，性能比各种树莓派~~香蕉派山里红派~~强到不知道哪里去了。不要用VPS，这已经不是作死了，简直是作大死。
本文演示用的是一台虚拟机，`Vmware Player` 就足够了，`VirtualBox` 更好，自带`NAT转发`的功能，将虚拟机的侦听端口映射到物理机的网络。 在这台虚拟机上，我为其配置了两块网卡。 ![](https://cdn.beijing2b.com/wp-content/uploads/2018/10/har-dvvare-options-memor-y-processor-s-hard-dis.png)
这台虚拟机安装CentOS7，既作为VPN客户端接入组织网络，又作为野生VPN服务器接入我的VPN客户端。
![](https://cdn.beijing2b.com/wp-content/uploads/2018/10/machine-generated-alternative-text-ens33-vpnz.png)
作为一个最小安装的CentOS7 , 还需要再装一些包。

```bash
sudo yum install vim
sudo yum install bind-utils
sudo yum install net-tools
sudo yum install openconnect
```
配置网卡`ens34` ,设置为静态IP地址。

```bash
sudo nano /etc/sysconfig/network-scripts/ifcfg-ens34
```

参考以下内容
```bash
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY=no
BOOTPROTO="static"
DEFROUTE="yes"
IPV4\_FAILURE\_FATAL="no"
IPV6INIT="no"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6\_FAILURE\_FATAL="no"
IPV6\_ADDR\_GEN_MODE="stable-privacy"
NAME="ens34"
UUID="7e9ac88d-ab2a-467a-a560-e3f4a92d55d2"
DEVICE="ens34"
ONBOOT="yes"
IPADDR=192.168.1.33
PREFIX=24
```
  配置网卡`ens33` ，设置为从DHCP获取，它将得到自动分配的IP地址/网关地址及DNS服务器地址
```bash
sudo nano /etc/sysconfig/network-scripts/ifcfg-ens33
```
参考以下内容
```bash
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="dhcp"
DEFROUTE="yes"
IPV4\_FAILURE\_FATAL="no"
IPV6INIT="no"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6\_FAILURE\_FATAL="no"
IPV6\_ADDR\_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="263bb93c-8338-4084-a542-7ca6cf8e4ae4"
DEVICE="ens33"
ONBOOT="yes"
```
查看网卡信息
```bash
ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens33: <BROADCAST,MULTICAST,UP,LOWER\_UP> mtu 1500 qdisc pfifo\_fast state UP mode DEFAULT group default qlen 1000
link/ether 00:0c:29:ab:cd:ef brd ff:ff:ff:ff:ff:ff
3: ens34: <BROADCAST,MULTICAST,UP,LOWER\_UP> mtu 1500 qdisc pfifo\_fast state UP mode DEFAULT group default qlen 1000
link/ether 00:0c:29:12:34:56 brd ff:ff:ff:ff:ff:ff
```
查看IP地址
```bash
ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
inet 127.0.0.1/8 scope host lo
valid\_lft forever preferred\_lft forever
inet6 ::1/128 scope host
valid\_lft forever preferred\_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER\_UP> mtu 1500 qdisc pfifo\_fast state UP group default qlen 1000
link/ether 00:0c:29:ab:cd:ef brd ff:ff:ff:ff:ff:ff
inet 192.168.137.185/24 brd 192.168.137.255 scope global noprefixroute dynamic ens33
valid\_lft 1714sec preferred\_lft 1714sec
3: ens34: <BROADCAST,MULTICAST,UP,LOWER\_UP> mtu 1500 qdisc pfifo\_fast state UP group default qlen 1000
link/ether 00:0c:29:12:34:56 brd ff:ff:ff:ff:ff:ff
inet 192.168.1.33/24 brd 192.168.1.255 scope global noprefixroute ens34
valid\_lft forever preferred\_lft forever
inet6 fe80::20c:29ff:fe70:be89/64 scope link
valid\_lft forever preferred\_lft forever
```
查看路由
```bash
ip route show
default via 192.168.137.2 dev ens33 proto dhcp metric 102
192.168.1.0/24 dev ens34 proto kernel scope link src 192.168.1.33 metric 101
192.168.137.0/24 dev ens33 proto kernel scope link src 192.168.137.185 metric 102
```
允许IPV4 转发
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```
欲永久生效需新建一个文件并编辑
```bash
sudo vi /etc/sysctl.d/vpn-server.conf
```
加入下面内容
```bash
# added for ipv4 forwading
net.ipv4.ip_forward=1
```
重启Linux  

### 安装ocserv

> 注：`arm64`的包依赖有问题，买了电视盒子/派的需要自己手动解决，本次演示用的`x64`则无此烦恼。
```bash
sudo yum install epel-release
sudo yum install ocserv
```
创建工作文件夹
```bash
mkdir ~/vpnc && cd ~/vpnc
```
 

#### 创建CA证书

> 注：自己有域名有数字证书的可以跳过这步。

创建野生CA
```bash
certtool --generate-privkey --outfile myca-key.pem

nano myca.tmpl
```
编辑模板加入以下内容
```bash
cn = "My VPN CA"
organization = "a VPN company"
serial = 1
expiration_days = 3650
ca
signing_key
cert_signing_key
crl_signing\key
```
生成野生CA
```bash
certtool --generate-self-signed --load-privkey myca-key.pem --template myca.tmpl --outfile myca-cert.pem
```
用野生CA给自己的服务器颁发证书
```bash
certtool --generate-privkey --outfile myvpn-key.pem

vi myvpn.tmpl
```
编辑模板加入以下内容
```bash
cn = "My VPN"
organization = "a VPN company"
serial = 2
expiration_days = 3650
encryption_key
signing_key
tls\_www\_server

certtool --generate-certificate --load-privkey myvpn-key.pem --load-ca-certificate myca-cert.pem --load-ca-privkey myca-key.pem --template myvpn.tmpl --outfile myvpn-cert.pem
```
服务器证书文件放进相应文件夹，放哪里自己记得就好，下面要用
```bash
sudo cp ~/vpnc/vpn_beijing2b_com.crt /etc/pki/ocserv/public/vpn_beijing2b_com.crt
sudo cp ~/vpnc/vpn_beijing2b_com.key /etc/pki/ocserv/private/vpn_beijing2b_com.key
```
#### 修改配置文件
```bash
sudo nano /etc/ocserv/ocserv.conf
```
参考以下内容，分配给野生VPN客户的网段不要与现有的冲突，最后一行`route`为组织内部IP网段，有多个网段且无法聚合的可以加入多行。
```bash
auth = "plain\[passwd=/etc/ocserv/\]"
listen-host = 192.168.1.33
tcp-port = 443
udp-port = 443
run-as-user = ocserv
run-as-group = ocserv
socket-file = ocserv.sock
chroot-dir = /var/lib/ocserv
isolate-workers = true
max-clients = 16
max-same-clients = 2
keepalive = 32400
dpd = 90
mobile-dpd = 1800
switch-to-tcp-timeout = 25
try-mtu-discovery = false
server-cert = /etc/pki/ocserv/public/vpn\_beijing2b\_com.crt
server-key = /etc/pki/ocserv/private/vpn\_beijing2b\_com.key
ca-cert = /etc/pki/ocserv/cacerts/ca.crt
cert-user-oid = 0.9.2342.19200300.100.1.1
tls-priorities = "NORMAL:%SERVER_PRECEDENCE:%COMPAT:-VERS-SSL3.0"
auth-timeout = 240
min-reauth-time = 300
max-ban-score = 50
ban-reset-time = 300
cookie-timeout = 300
deny-roaming = false
rekey-time = 172800
rekey-method = ssl
use-occtl = true
pid-file = /var/run/ocserv.pid
device = mytun
predictable-ips = true
default-domain = YourOrg.com
ipv4-network = 192.168.2.0
ipv4-netmask = 255.255.255.0
dns = 192.168.1.33
ping-leases = false
cisco-client-compat = true
dtls-legacy = true
user-profile = profile.xml
route = 10.0.0.0/255.0.0.0
```
创建野生VPN用户
```bash
sudo ocpasswd -c /etc/ocserv/ocpasswd myvpnuser
```
配置防火墙
```bash
sudo systemctl start firewalld.service
sudo firewall-cmd --permanent --zone=public --add-port=443/tcp
sudo firewall-cmd --permanent --zone=public --add-port=443/udp
sudo firewall-cmd --permanent --add-masquerade
sudo firewall-cmd --permanent --direct --passthrough ipv4 -t nat -A POSTROUTING -o ens34 -j MASQUERADE
sudo firewall-cmd --reload
```
拨入组织VPN，假设用的是用户名密码验证的话，就把密码存到文件`/etc/openconnect/password`里
```bash
echo $(sudo cat /etc/openconnect/password) | sudo openconnect -b https://YourORGVPNURL -u YourOrgVPNUsername --passwd-on-stdin
```
接入之后再看网卡，多了一块`tun0` ，注意名字可能不同
```bash
ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens33: <BROADCAST,MULTICAST,UP,LOWER\_UP> mtu 1500 qdisc pfifo\_fast state UP mode DEFAULT group default qlen 1000
link/ether 00:0c:29:70:be:7f brd ff:ff:ff:ff:ff:ff
3: ens34: <BROADCAST,MULTICAST,UP,LOWER\_UP> mtu 1500 qdisc pfifo\_fast state UP mode DEFAULT group default qlen 1000
link/ether 00:0c:29:70:be:89 brd ff:ff:ff:ff:ff:ff
6: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER\_UP> mtu 1406 qdisc pfifo\_fast state UNKNOWN mode DEFAULT group default qlen 500
link/none
```
看到流量已经全都去往组织内部网络
```bash
ip route show
default dev tun0 scope link
default via 192.168.137.2 dev ens33 proto dhcp metric 102
10.x.y.0/24 dev tun0 scope link
a.b.c.d via 192.168.137.2 dev ens33 src 192.168.137.187
192.168.1.0/24 dev ens34 proto kernel scope link src 192.168.1.33 metric 101
192.168.137.0/24 dev ens33 proto kernel scope link src 192.168.137.187 metric 102
```
接下来测试下野生VPN服务
```bash
sudo ocserv -c /etc/ocserv/ocserv.conf -f -d 1
```
从客户端发起连接，使用思科官方的`AnyConnect` 连接到`192.168.1.33`，用户名就是刚才创建在`/etc/ocserv/ocpasswd`里面的， 另外别忘记去掉对野生证书的阻挡。   ![VPN n Start VPN when AnyConnect is started n Minimize AnyConnect on VPN connect n Allow local (LAN) access when using VPN (if configured) Disable Captive Portal Detection glock connections to untrusted serve ](https://cdn.beijing2b.com/wp-content/uploads/2018/10/vpn-n-start-vpn-when-anyconnect-is-started-n-m.png)
在客户端上运行`netstat -nr`  或者`route print`  查看路由，注意下图高亮的一行。   ![](https://cdn.beijing2b.com/wp-content/uploads/2018/10/netstat.jpg) ![AnyConnect VPN Virtual Private Network (VPN) Statistics Route Details Connection Information State: Tunnel Mode (IPv4): Tunnel Mode (IPv6): Dynamic Tunnel Exclusion: Dynamic Tunnel Inclusion: Duration: Session Disconnect: Address Information Client (IPv4): Client (IPv6): Server: Bytes Sent: Received: Frames Sent: Firewall Message History Connected Split Include Drop All Traffic None None None 192.168.2.78 Not Available 192.168.1.33 20971 10814B 170 Reset Export Stats...](https://cdn.beijing2b.com/wp-content/uploads/2018/10/anyconnect-vpn-virtual-private-network-vpn-sta.png) 接下来可以将`ocserv`设为开机自动启动
```bash
sudo systemctl enable ocserv
```
 

### DNS

`DNS conditional-fowarding`，条件转发 因为组织内部网络的DNS记录只能通过组织内部的DNS服务器解析，外部的公网记录它虽然能解析，但是并不优化——往往给你一个组织所使用的ISP优化的结果，而且我家自己有一个DNS服务器，`192.168.1.30` ，每天从“你懂的”列表更新。 于是我要做到，凡是查询组织内部的DNS记录就转发到组织的DNS服务器，其他就转发到我自己的DNS服务器。 
野生服务器安装`bind  DNS`
```bash
sudo yum install bind
```
备份原始配置文件
```bash
sudo cp /etc/named.conf /etc/named.conf.ori
```
编辑配置文件
```bash
sudo vi /etc/named.conf
```
内容改为下面的，组织有多个顶级域名的，就添加多个`zone`.
```bash
options {
listen-on port 53 { 192.168.1.33; };
\# listen-on-v6 port 53 {none; };
directory "/var/named";
dump-file "/var/named/data/cache_dump.db";
statistics-file "/var/named/data/named_stats.txt";
memstatistics-file "/var/named/data/named\_mem\_stats.txt";
allow-query { localhost;192.168.1.0/24; };
recursion yes;
forwarders {
192.168.1.30;
};
forward only;
dnssec-enable no;
dnssec-validation no;
/\* Path to ISC DLV key */
bindkeys-file "/etc/named.iscdlv.key";
managed-keys-directory "/var/named/dynamic";
pid-file "/run/named/named.pid";
session-keyfile "/run/named/session.key";
};
logging {
channel default_debug {
file "data/named.run";
severity dynamic;
};
};
zone "yourORG.com" {
type forward;
forward only;
forwarders {10.xx.yy.zz; 10.aa.bb.cc };
};
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```
检查配置
```bash
sudo named-checkconf
```
DNS服务加入开机启动
```bash
sudo systemctl enable named
sudo systemctl start named
```
为DNS服务配置防火墙
```bash
sudo firewall-cmd --permanent --add-port=53/tcp
sudo firewall-cmd --permanent --add-port=53/udp
sudo firewall-cmd --reload
```
在客户端上尝试查询组织内部DNS记录，注意第一次使用的是我家DNS服务器，查不到结果（废话），第二次则成功。 ![dig , DiG 9.8.3-P1 ; ; global options: +cmd • Got answer: —o com @192.168.1.30 com @192.168.1.30 , opcode: QUERY, status: NOERROR, id: 5ø78 • flags: qr rd ra; QUERY: 1, ANSWER: o, AUTHORITY: 1, ADDITIONAL: ø QUESTION SECTION: AUTHORITY SECTION: . com. 752 3øø 3øø 24192øø gøø Query time: 604 msec 299 IN IN SOA '"'t....com . com Icom. com. 2014093 SERVER: WHEN: Mon Oct 1 15:13:15 2018 MSG SIZE rcvd: løl dig , DiG 9.8.3-P1 ; ; global options: +cmd • Got answer: , opcode: QUERY, status: NOERROR, id: 18ø3 • flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: o, ADDITIONAL: ø QUESTION SECTION: ANSWER _SECTION: . com. Query time: msec 228 IN IN 10 SERVER: WHEN: Mon Oct 1 15:13:26 2018 MSG SIZE rcvd: 5ø ](https://cdn.beijing2b.com/wp-content/uploads/2018/10/dig-dig-9-8-3-p1-global-options-cmd-g.png)  

### 及时推送vpn断开信息

有时候从野生VPN到组织官方VPN断了，还不知道的话就傻傻等，我们可以用`Slack` 接收相关消息。
先去创建一个机器人 ![@ https://api.slack.com/apps here g Slack apps updates actices aeprints eatures Il integrations ng webhooks ommands g bots Channels rise Grid Documentation Your Apps Build something amazing. Use our APIs to build an app that makes people's working lives better. You can create an app that's just for your workspace or create a public Slack App to list in the App Directory, where anyone on Slack can discover it. Create an App Don't see an app you're looking for? Sign in to another workspace. ](https://cdn.beijing2b.com/wp-content/uploads/2018/10/https-api-slack-com-apps-here-g-slack-apps.png)
起个名字 ![Create a Slack App Interested in the next generation of apps? x We're improving app development and distribution. Join the API Preview period for workspace tokens and the Permissions API. App Name VPN watch cat Don't worry; you'll be able to change this later. Development Slack Workspace Beijing2b Your app belongs to this workspace—leaving this workspace will remove your ability to manage this app. Unfortunately, this can't be changed later. By creating a Web API Application, you agree to the Slack API Terms of Service. Create App ](https://cdn.beijing2b.com/wp-content/uploads/2018/10/create-a-slack-app-interested-in-the-next-generat.png)
![Basic Information Building Apps for Slack Create an app that's just for your workspace (or build one that can be used by any workspace) by following the steps below. Add features and functionality Choose and configure the tools you'll need to create your app (or review all our documentation). Incoming Webhooks Post messages from external sources into Slack. Slash Commands Allow users to perform app actions by typing commands in Slack. Bots Add a bot to allow users to exchange messages with your app. Interactive Components Add buttons to your app's messages, and create an interactive experience for users. Event Subscriptions Make it easy for your app to respond to activity in Slack. Permissions Configure permissions to allow your app to interact with the Slack API. Install your app to your workspace Manage distribution ](https://cdn.beijing2b.com/wp-content/uploads/2018/10/basic-information-building-apps-for-slack-create.png)
![VPN watch cat Settings Basic Information Collaborators Install App Manage Distribution Features Incoming Webhooks Interactive Components Slash Commands OAuth & Permissions Event Subscriptions Bot Users user ID Translation Slack Help Contact Policies Our Blog Incoming Webhooks Activate Incoming Webhooks Coo Incoming webhooks are a simple way to post messages from external sources into Slack. They make use of normal HTTP requests with a JSON payload, which includes the message and a few other optional details. You can include message attachments to display richly-formatted messages. Each time your app is installed, a new Webhook URL will be generated. If you deactivate incoming webhooks, new Webhook URLs will not be generated when your app is installed to your team. If you'd like to remove access to existing Webhook URLs, you will need to Revoke All OAuth Tokens. Webhook URLs for Your Workspace To dispatch messages with your webhook URL, send your message in JSON as the body of an POST request. Add this webhook to your workspace below to activate this curl example. Sample curl request to post to a channel: curl -X POST -H 'Content-type: application/json' --data Webhook URL No webhooks have been added yet. Add New Webhook to Workspace ' {"text" World! Channel Added By ](https://cdn.beijing2b.com/wp-content/uploads/2018/10/vpn-watch-cat-settings-basic-information-collab.png)
URL记下来，待会要用 ![Incoming Webhooks Activate Incoming Webhooks Incoming webhooks are a simple way to post messages from external sources into Slack. They make use of normal HTTP requests with a JSON payload, which includes the message and a few other optional details. You can include message attachments to display richly-formatted messages. Each time your app is installed, a new Webhook URL will be generated. If you deactivate incoming webhooks, new Webhook URLs will not be generated when your app is installed to your team. If you'd like to remove access to existing Webhook URLs, you will need to Revoke All OAuth Tokens. Webhook URLs for Your Workspace To dispatch messages with your webhook URL, send your message in JSON as the body of an POST request. Add this webhook to your workspace below to activate this curl example. Sample curl request to post to a channel: curl -X POST -H 'Content-type: application/json' --data ' {"text" World! ' https://hooks.slack.com/services.••.......'.. • Webhook URL https://hooks.slack.com/services, Add New Webhook to Workspace Channel slackbot Added By Yu Oct 3, 2018 ](https://cdn.beijing2b.com/wp-content/uploads/2018/10/incoming-webhooks-activate-incoming-webhooks-inc.png)
下面命令把断开时的日志发到Slack ，可以与拨号去组织VPN的命令写在一个shell 脚本里
```bash
tail -f /var/log/messages | grep --line-buffered "(tun0): state change: activated -> unmanaged" | xargs -I @ curl -s \
curl -X POST \
-H 'Content-type: application/json' \
--data '{"username":"curl","text":"@"}' \
https://hooks.slack.com/services/XXX/YYY/ZZZ
```
最后效果如下图： ![](https://cdn.beijing2b.com/wp-content/uploads/2018/10/slackbot-active-slackbot-slack-beijing2b-o.png)
因为我不怎么会Linux，所以也不知道有木有更优化的办法，就到这里吧。

<center>感谢观看，保留所有权利，未经许可不得转载。</center>