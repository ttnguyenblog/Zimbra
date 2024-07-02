# Hệ Điều Hành CentOS 7:
- **Cài Đặt Email Server Zimbra** (có hướng dẫn cài trong bài báo cáo)
- **Cài Đặt IPv4 Address**: 192.168.1.10
- **Subnet Mask**: 255.255.255.0
- **Default Gateway**: 192.168.1.1

# Hệ Điều Hành Windows 7:
- **Cài Đặt IPv4 Address**: 192.168.1.20
- **Subnet Mask**: 255.255.255.0
- **Default Gateway**: 192.168.1.1

# Zimbra Server: 
- Tạo 2 tài khoản U1, U2
- Truy cập 192.168.1.10:7071/ZimbrAdmin
- Truy cập 192.168.1.10 vào tài khoản client

```bash
yum install xrdp
systemctl enable xrdp
adduser xrdp ssl-cert
systemctl start xrdp

# Cài đặt giao diện Centos:

yum update
yum groupinstall "GNOME Desktop" "Graphical Administration Tools"
systemctl set-default graphical

*Chỉnh sửa file /etc/selinux/config:

gedit /etc/selinux/config
SELINUX=disabled

sestatus
SELinux status: disabled

*Dừng và gỡ bỏ postfix:

systemctl stop postfix
yum remove postfix -y

hostnamectl set-hostname mail.zim.local

yum update -y ; reboot

*Chỉnh sửa file /etc/hosts:

gedit /etc/hosts

ip local mail.zim.local mail

gedit /etc/resolv.conf
nameserver ip local

*Cài đặt các gói cần thiết:

yum install dnf

dnf install nano wget bind bind-utils telnet perl firewalld nmap-ncat -y

gedit /etc/named.conf


forwarders {
	8.8.8.8;
	8.8.4.4;
};

zone "zim.local" { 
	type master;
	file "/var/named/zim.local.hosts"; 
};

//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//
// See the BIND Administrator's Reference Manual (ARM) for details about the
// configuration located in /usr/share/doc/bind-{version}/Bv9ARM.html

options {
	listen-on port 53 { any; };
	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	recursing-file  "/var/named/data/named.recursing";
	secroots-file   "/var/named/data/named.secroots";
	allow-query     { any; };

	/* 
	 - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
	 - If you are building a RECURSIVE (caching) DNS server, you need to enable 
	   recursion. 
	 - If your recursive DNS server has a public IP address, you MUST enable access 
	   control to limit queries to your legitimate users. Failing to do so will
	   cause your server to become part of large scale DNS amplification 
	   attacks. Implementing BCP38 within your network would greatly
	   reduce such attack surface 
	*/
	recursion yes;

	dnssec-enable yes;
	dnssec-validation yes;

	/* Path to ISC DLV key */
	bindkeys-file "/etc/named.root.key";

	managed-keys-directory "/var/named/dynamic";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
	
	forwarders {
		8.8.8.8;
		8.8.4.4;
	};
	
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
	type hint;
	file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

zone "zim.local" { 
	type master;
	file "/var/named/zim.local.hosts"; 
};

gedit /var/named/zim.local.hosts
$ttl 38400
zim.local.   IN  SOA  mail.zim.local. admin.zim.local. (
                       1520401032
                       10800
                       3600
                       604800
                       38400 )
zim.local.   IN  NS   mail.zim.local.
mail.zim.local.  IN  A  192.168.30.143
zim.local.   IN  MX  10 mail

systemctl enable named
systemctl restart named

cd /opt/
wget https://files.zimbra.com/downloads/8.8.15_GA/zcs-8.8.15_GA_3953.RHEL8_64.20200629025823.tgz

tar -zxvf zcs-8.8.15_GA_3953.RHEL8_64.20200629025823.tgz
cd zcs-8.8.15_GA_3953.RHEL8_64.20200629025823

./install.sh

./install.sh --platform-override


/opt/zimbra/libexec/zmdkimkeyutil-qd zim.local
/opt/zimbra/libexec/zmdkimkeyutil -ad zim.local -s mail

systemctl start firewalld

firewall-cmd --permanent --zone=public --add-port=25/tcp
firewall-cmd --permanent --zone=public --add-port=80/tcp
firewall-cmd --permanent --zone=public --add-port=110/tcp
firewall-cmd --permanent --zone=public --add-port=143/tcp
firewall-cmd --permanent --zone=public --add-port=443/tcp
firewall-cmd --permanent --zone=public --add-port=465/tcp
firewall-cmd --permanent --zone=public --add-port=587/tcp
firewall-cmd --permanent --zone=public --add-port=993/tcp
firewall-cmd --permanent --zone=public --add-port=995/tcp
firewall-cmd --permanent --zone=public --add-port=3443/tcp
firewall-cmd --permanent --zone=public --add-port=5222/tcp
firewall-cmd --permanent --zone=public --add-port=5223/tcp
firewall-cmd --permanent --zone=public --add-port=9071/tcp
firewall-cmd --permanent --zone=public --add-port=8443/tcp
firewall-cmd --permanent --zone=public --add-port=7071/tcp
firewall-cmd --permanent --zone=public --add-port=53/tcp
firewall-cmd --permanent --zone=public --add-port=53/udp

firewall-cmd --reload 



