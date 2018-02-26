静态ip

# vim /etc/sysconfig/network-scripts/ifcfg-eth0

```bash
HWADDR="00:15:5D:07:F1:02"
TYPE="Ethernet"
BOOTPROTO="static" #dhcp改为static
DEFROUTE="yes"
PEERDNS="yes"
PEERROUTES="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_PEERDNS="yes"
IPV6_PEERROUTES="yes"
IPV6_FAILURE_FATAL="no"
NAME="eth0"
UUID="bb3a302d-dc46-461a-881e-d46cafd0eb71"
ONBOOT="yes" #开机启用本配置
IPADDR=192.168.7.106 #静态IP
GATEWAY=192.168.7.1 #默认网关
NETMASK=255.255.255.0 #子网掩码
DNS1=192.168.7.1 #DNS 配置
```


service network restart

ip addr



yum update


yum install vsftpd

chkconfig vsftpd on


管理vsftpd相关命令：

停止vsftpd:  service vsftpd stop

重启vsftpd:  service vsftpd restart


