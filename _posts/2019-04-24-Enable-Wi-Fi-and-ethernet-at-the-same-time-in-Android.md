---
layout: post
title:  "Enable Wi-Fi and ethernet at the same time in Android"
date:   2019-04-24 09:50:00 +0800
---

# What's the problem?
Nowadays, I received a feature request: enabale Wi-Fi and ethernet at the same time in one Android device.
However, in most Android devices I know, the **priority** of eth0 is greater than wlan0. In another word, if you plug in eth0 after wlan0 linked, the wlan0 would not work any longer. Here is a trail for verify this point:

## Trail A: verify original AOSP network mechanism:
### eth0 down, wlan0 up
First of all, keep eth0 disconnected and connected to a Wi-Fi:
~~~~
device:/ $ ifconfig
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope: Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:0 TX bytes:0

wlan0     Link encap:Ethernet  HWaddr 04:04:04:04:04:04
          inet addr:192.168.1.5  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::6e6:76ff:febc:88f5/64 Scope: Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:52 errors:0 dropped:0 overruns:0 frame:0
          TX packets:46 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:6805 TX bytes:4917

eth0      Link encap:Ethernet  HWaddr aa:aa:aa:aa:aa:aa
          inet6 addr: fe80::a852:4bff:fee8:9d9b/64 Scope: Link
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:3914 errors:0 dropped:0 overruns:0 frame:0
          TX packets:54 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:516728 TX bytes:5983
          Interrupt:24
device:/ $ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=37 time=251 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=37 time=39.2 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=37 time=40.4 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=37 time=42.4 ms
device:/ $ ping 192.168.1.1
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=6.12 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=64 time=10.6 ms
64 bytes from 192.168.1.1: icmp_seq=3 ttl=64 time=5.10 ms
~~~~


### eth0 up, wlan0 up
~~~~
device:/ $ ifconfig
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope: Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:0 TX bytes:0

wlan0     Link encap:Ethernet  HWaddr 04:04:04:04:04:04
          inet addr:192.168.1.5  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::6e6:76ff:febc:88f5/64 Scope: Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:82 errors:0 dropped:0 overruns:0 frame:0
          TX packets:82 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:8962 TX bytes:7437

eth0      Link encap:Ethernet  HWaddr aa:aa:aa:aa:aa:aa
          inet addr:172.31.23.13  Bcast:172.31.23.255  Mask:255.255.255.0
          inet6 addr: fe80::a852:4bff:fee8:9d9b/64 Scope: Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:3927 errors:0 dropped:0 overruns:0 frame:0
          TX packets:63 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:520039 TX bytes:7309
          Interrupt:24
device:/ $ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=110 time=194 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=110 time=181 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=110 time=198 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=111 time=171 ms
^C
device:/ $ ping 192.168.1.1
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
^C
--- 192.168.1.1 ping statistics ---
6 packets transmitted, 0 received, 100% packet loss, time 5008ms
device:/ $ tracepath 8.8.8.8
 1?: [LOCALHOST]                                         pmtu 1500
 1:  172.31.23.1                                           2.900ms
 1:  172.31.23.1                                           2.838ms
 2:  172.31.255.69                                         1.470ms
 3:  172.31.255.1                                          1.507ms
device:/ $ tracepath 192.168.1.1
 1?: [LOCALHOST]                                         pmtu 1500
 1:  172.31.23.1                                           3.344ms
 1:  172.31.23.1                                           2.658ms
 2:  172.31.255.69                                         1.759ms
 3:  172.31.255.1                                          1.410ms
 4:  172.21.255.89                                         1.459ms
device:/ $ ip route
172.31.23.0/24 dev eth0  proto kernel  scope link  src 172.31.23.13
192.168.1.0/24 dev wlan0  proto kernel  scope link  src 192.168.1.5
~~~~
As we can see, kernel will visit wlan0 even though I want to `tracepath 192.168.1.1`


# Solution

## a *temporary* solution:
~~~~
device:/ # ip rule flush
device:/ # ip rule add pref 1000
device:/ # ip rule
0:      from all lookup local
1000:   from all lookup main
device:/ # ip route add default via 172.31.23.0
device:/ # ip route
default via 172.31.23.0 dev eth0
172.31.23.0/24 dev eth0  proto kernel  scope link  src 172.31.23.13
192.168.1.0/24 dev wlan0  proto kernel  scope link  src 192.168.1.5
device:/ # tracepath 192.168.1.2
 1?: [LOCALHOST]                                         pmtu 1500
 1:  192.168.1.2                                          21.242ms reached
 1:  192.168.1.2                                           9.880ms reached
device:/data # busybox wget http://192.168.1.2        # test HTTP protocol
Connecting to 192.168.1.2 (192.168.1.2:80)
index.html           100% |*******************************|   156   0:00:00 ETA
device:/data # rm index.html
device:/data # busybox wget http://172.21.158.133     # test HTTP protocol
Connecting to 172.21.158.133 (172.21.158.133:80)
Connecting to 172.21.158.133 (172.21.158.133:80)
index.html           100% |*******************************| 11046   0:00:00 ETA
~~~~

PS: we can ask kernel which interface will be used to send a packet to a specific ip address by `ip route get` command like this: `ip route get 8.8.8.8`



## What's next?
However, the solution above is very temporary. Android's java program like Connectivity inside of `framework.jar` and `netd` native program will affect Linux kernel via `libnetlink` which is very very complicated... Maybe I'll design a solution that was compatible with static/DHCP IP and Wi-Fi hotspot mode.





------
*remark: some information was modified like HWaddr and hostname*
