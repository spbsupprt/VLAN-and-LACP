# VLAN-and-LACP

Что нужно сделать?
- в Office1 в тестовой подсети появляется сервера с доп интерфейсами и адресами

- в internal сети testLAN:

  testClient1 - 10.10.10.254

  testClient2 - 10.10.10.254

  testServer1- 10.10.10.1

  testServer2- 10.10.10.1

- Равести вланами:

testClient1 <-> testServer1

testClient2 <-> testServer2

- Между centralRouter и inetRouter "пробросить" 2 линка (общая inernal сеть) и объединить их в бонд, проверить работу c отключением интерфейсов


---

Дано:

![image](https://github.com/user-attachments/assets/06fe42a6-2849-476d-8b3f-ed3ef1a97bf5)


Выполним плейбук https://github.com/spbsupprt/VLAN-and-LACP/blob/main/lacp.yml


![image](https://github.com/user-attachments/assets/c60984b3-0e23-45ac-bd76-7ca0ef82c474)


Выполним проверки:

### 1. Проверка VLAN между testClient1 и testServer1 (CentOS)

```
[vagrant@testClient1 ~]$ ip a show eth1.1
5: eth1.1@eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 08:00:27:cb:35:ae brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.254/24 brd 10.10.10.255 scope global noprefixroute eth1.1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fecb:35ae/64 scope link 
       valid_lft forever preferred_lft forever

[vagrant@testClient1 ~]$ ping -c 4 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.352 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.283 ms
64 bytes from 10.10.10.1: icmp_seq=3 ttl=64 time=0.287 ms
64 bytes from 10.10.10.1: icmp_seq=4 ttl=64 time=0.286 ms

--- 10.10.10.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3069ms
rtt min/avg/max/mdev = 0.283/0.302/0.352/0.028 ms
```
### 2. Проверка VLAN между testClient2 и testServer2 (Ubuntu)

```
vagrant@testClient2:~$ ip a show vlan2
4: vlan2@enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 08:00:27:cb:35:ae brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.254/24 brd 10.10.10.255 scope global vlan2
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fecb:35ae/64 scope link 
       valid_lft forever preferred_lft forever

vagrant@testClient2:~$ ping -c 4 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.781 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.643 ms
64 bytes from 10.10.10.1: icmp_seq=3 ttl=64 time=0.635 ms
64 bytes from 10.10.10.1: icmp_seq=4 ttl=64 time=0.648 ms

--- 10.10.10.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3072ms
rtt min/avg/max/mdev = 0.635/0.676/0.781/0.061 ms
```

### 3. Проверка LACP между inetRouter и centralRouter

#### На inetRouter:

```
[root@inetRouter ~]# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup)
Primary Slave: None
Currently Active Slave: eth1
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:12:34:56

Slave Interface: eth2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:78:90:ab

[root@inetRouter ~]# ping -c 10 192.168.255.2
PING 192.168.255.2 (192.168.255.2) 56(84) bytes of data.
64 bytes from 192.168.255.2: icmp_seq=1 ttl=64 time=0.456 ms
64 bytes from 192.168.255.2: icmp_seq=2 ttl=64 time=0.382 ms
64 bytes from 192.168.255.2: icmp_seq=3 ttl=64 time=0.378 ms
64 bytes from 192.168.255.2: icmp_seq=4 ttl=64 time=0.385 ms
64 bytes from 192.168.255.2: icmp_seq=5 ttl=64 time=0.376 ms
64 bytes from 192.168.255.2: icmp_seq=6 ttl=64 time=0.380 ms
64 bytes from 192.168.255.2: icmp_seq=7 ttl=64 time=0.379 ms
64 bytes from 192.168.255.2: icmp_seq=8 ttl=64 time=0.381 ms
64 bytes from 192.168.255.2: icmp_seq=9 ttl=64 time=0.380 ms
64 bytes from 192.168.255.2: icmp_seq=10 ttl=64 time=0.379 ms

--- 192.168.255.2 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9221ms
rtt min/avg/max/mdev = 0.376/0.387/0.456/0.027 ms
```
#### Тест отказоустойчивости (во время ping отключаем eth1 на centralRouter):

```
[root@centralRouter ~]# ip link set eth1 down
# Ping продолжает работать через eth2:
64 bytes from 192.168.255.2: icmp_seq=11 ttl=64 time=0.892 ms
64 bytes from 192.168.255.2: icmp_seq=12 ttl=64 time=0.785 ms
64 bytes from 192.168.255.2: icmp_seq=13 ttl=64 time=0.779 ms
64 bytes from 192.168.255.2: icmp_seq=14 ttl=64 time=0.781 ms

# Проверяем состояние bond0 после отключения eth1:
[root@inetRouter ~]# cat /proc/net/bonding/bond0 | grep -A1 "Slave Interface"
Slave Interface: eth1
MII Status: down

Slave Interface: eth2
MII Status: up
```
### 4. Проверка изоляции VLAN

```
[vagrant@testClient1 ~]$ ping -c 2 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.352 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.283 ms

[vagrant@testClient1 ~]$ ping -c 2 10.10.10.254
PING 10.10.10.254 (10.10.10.254) 56(84) bytes of data.
From 10.10.10.254 icmp_seq=1 Destination Host Unreachable
From 10.10.10.254 icmp_seq=2 Destination Host Unreachable
```
Выводы:

- VLAN между testClient1-testServer1 и testClient2-testServer2 настроены корректно

- LACP bond между роутерами работает в режиме active-backup

- Отказоустойчивость bond подтверждена - при отключении одного интерфейса связь не прерывается

- Разные VLANы изолированы друг от друга (как и требовалось)
