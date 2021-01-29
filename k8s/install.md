#安装k8s
##前期准备
###step 1 关闭selinux
	sed -i 's/SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
	reboot
  
###step 2 检查系统内核版本,要求3.8以上
	uanme -a
  
###step 3 关闭防火墙
	systemctl stop firewalld
###step 4 安装yum源,并安装必要的软件
	curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
	yum install wget net-tools telnet tree nmap sysstat lrzsz dos2unix bind-utils -y
##在dns服务器上安装dns服务
  安装bind9
###step 1
  	yum install bind -y
###step 2 修改配置文件
  	sed -i 's/listen-on port 53.*/listen-on port 53 { 0.0.0.0; };/g' /etc/named.conf
  	sed -i 's/allow-query.*/allow-query     { any; };/g' /etc/named.conf
  	sed -i "/allow-query/a\\\tforwarders\\t{ 10.4.7.254; };" /etc/named.conf   #10.4.7.254 为上级网关地址
  	sed -i 's/dnssec-enable.*/dnssec-enable no;/g' /etc/named.conf
  	sed -i 's/dnssec-validation.*/dnssec-validation no;/g' /etc/named.conf
  	named-checkconf   #检测配置
###step 3添加动态域
在/etc/named.rfc1912.zones 中添加

	zone "host.com" IN {
        type master;
        file "host.com.zone";
        allow-update { 10.4.7.11; };
	};
	zone "od.com" IN {
        type master;
        file "od.com.zone";
        allow-update { 10.4.7.11; };
	};
###step 4添加/var/named/host.com.zone
	
	$ORIGIN	host.com.
	$TTL	600; 10 minutes
	@	IN	SOA	dns.host.com. dnsadmin.host.com. (
					2021012801	; serial
					10800		; refresh (3 hours)
					900			; retry (15 minutes)
					604800		; expire (1 week)
					86400		; minimum (1 day)
					)
			NS	dns.host.com.
	$TTL	60 ; 1 minute
	dns	A	10.4.7.11
	HDSS7-11	A	10.4.7.11
	HDSS7-12	A	10.4.7.12
	HDSS7-21	A	10.4.7.21
	HDSS7-22	A	10.4.7.22
	HDSS7-200	A	10.4.7.200


###step 5 vi /var/named/od.com.zone
	$ORIGIN od.com.
	$TTL 600  ; 10 minutes
	@	IN	SOA	dns.od.com. dnsadmin.od.com. (
					2021012801      ; serial
					10800           ; refresh (3 hours)
					900             ; retry (15 minutes)
					604800          ; expire (1 week)
					86400           ; minimum (1 day)
					)
			NS	dns.od.com.
	$TTL	60	; 1 minute
	dns	A	10.4.7.11
