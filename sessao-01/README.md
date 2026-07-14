### ip a

Interface principal: enp1s0
- IP: 172.30.1.2/24
- MAC: 4e:d1:6f:eb:e5:7d
- Estado: UP

Output completo:
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc fq_codel state UP group default qlen 1000
    link/ether 4e:d1:6f:eb:e5:7d brd ff:ff:ff:ff:ff:ff
    inet 172.30.1.2/24 brd 172.30.1.255 scope global dynamic noprefixroute enp1s0
       valid_lft 86307921sec preferred_lft 75518721sec
    inet6 fe80::a1fd:c8fd:a7c6:c90/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1454 qdisc noqueue state DOWN group default 
    link/ether 1a:fe:75:2d:fe:1f brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
```
### ss -tuln

Portas TCP em escuta (LISTEN):
- 22 (SSH) — acessível externamente (0.0.0.0 / [::])
- 53 (DNS) — apenas local (127.0.0.53/127.0.0.54)
- 39229, 40200, 40205, 40300, 40305 — serviços auxiliares do ambiente

Output completo:
root@ubuntu:~$ ss -tuln
Netid           State            Recv-Q            Send-Q                                           Local Address:Port                        Peer Address:Port           Process           
udp             UNCONN           0                 0                                                   127.0.0.54:53                               0.0.0.0:*                                
udp             UNCONN           0                 0                                                127.0.0.53%lo:53                               0.0.0.0:*                                
udp             UNCONN           0                 0                                                   172.30.1.2:68                               0.0.0.0:*                                
udp             UNCONN           0                 0                                            172.30.1.2%enp1s0:68                               0.0.0.0:*                                
udp             UNCONN           0                 0                            [fe80::a1fd:c8fd:a7c6:c90]%enp1s0:546                                 [::]:*                                
tcp             LISTEN           0                 4096                                                127.0.0.54:53                               0.0.0.0:*                                
tcp             LISTEN           0                 4096                                                 127.0.0.1:39229                            0.0.0.0:*                                
tcp             LISTEN           0                 4096                                                   0.0.0.0:22                               0.0.0.0:*                                
tcp             LISTEN           0                 511                                                    0.0.0.0:40205                            0.0.0.0:*                                
tcp             LISTEN           0                 128                                                    0.0.0.0:40200                            0.0.0.0:*                                
tcp             LISTEN           0                 4096                                             127.0.0.53%lo:53                               0.0.0.0:*                                
tcp             LISTEN           0                 4096                                                      [::]:22                                  [::]:*                                
tcp             LISTEN           0                 4096                                                         *:40305                                  *:*                                
tcp             LISTEN           0                 4096                                                         *:40300                                  *:*                                
root@ubuntu:~$


```
[cola aqui o output completo que enviaste]
```
