SECTION 1:
machine 1
=========
root@machine1:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: 10.1.1.1 <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:c7:92:54 brd ff:ff:ff:ff:ff:ff
    inet /24 brd 10.1.1.255 scope global dynamic noprefixroute eth0
       valid_lft 940sec preferred_lft 940sec
    inet6 fe80::bc5f:787a:7d7d:b544/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
root@machine1:~# 

machine 2
=========
root@machine2:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: 10.1.1.2 <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:38:77:51 brd ff:ff:ff:ff:ff:ff
    inet /24 brd 10.1.1.255 scope global dynamic noprefixroute eth0
       valid_lft 940sec preferred_lft 940sec
    inet6 fe80::9ce2:7e5:e158:470/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
root@machine2:~# 

root@machine1:~# ping 10.1.1.2
PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=0.433 ms
64 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=0.776 ms
64 bytes from 10.1.1.2: icmp_seq=3 ttl=64 time=1.70 ms
^C
--- 10.1.1.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2006ms
rtt min/avg/max/mdev = 0.433/0.969/1.699/0.534 ms
root@machine1:~# traceroute 10.1.1.2
traceroute to 10.1.1.2 (10.1.1.2), 64 hops max
  1   10.1.1.2  0.454ms  0.221ms  0.223ms 

root@machine2:~# ping 10.1.1.1
PING 10.1.1.1 (10.1.1.1) 56(84) bytes of data.
64 bytes from 10.1.1.1: icmp_seq=1 ttl=64 time=0.598 ms
64 bytes from 10.1.1.1: icmp_seq=2 ttl=64 time=4.24 ms
64 bytes from 10.1.1.1: icmp_seq=3 ttl=64 time=0.857 ms
64 bytes from 10.1.1.1: icmp_seq=4 ttl=64 time=0.436 ms
^C
--- 10.1.1.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3021ms
rtt min/avg/max/mdev = 0.436/1.531/4.236/1.568 ms
root@machine2:~# traceroute 10.1.1.1
traceroute to 10.1.1.1 (10.1.1.1), 64 hops max
  1   10.1.1.1  0.340ms  0.305ms  0.642ms 
root@machine2:~#

=============================================================
SECTION 2:
#In the first machine, block incoming traffic from second machine to ssh port 22.
#First make sure that traffic to port 22 is allowed on first machine. 
#Then disallow traffic from the second machine only.
iptables -A INPUT -i eth0 -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -i eth0 -p tcp --destination-port 22 -s 10.1.1.1 -j DROP

#add rules permanently:
sudo iptables-save

==============================================================
SECTION3:
#In the first machine, ensure traffic to port 12345 is accepted.
#Then forward this traffic back to the second machine on port 22.
iptables -A INPUT -i eth0 -p tcp --dport 12345 -j ACCEPT
iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 12345 -j DNAT --to 10.1.1.2:22
#add rules permanently:
sudo iptables-save
