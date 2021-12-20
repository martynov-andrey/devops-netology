# devops-netology

---

### Домашнее задание к занятию "3.7. Компьютерные сети, лекция 2"

1. #### Проверьте список доступных сетевых интерфейсов на вашем компьютере. Какие команды есть для этого в Linux и в Windows?

Для Linux:

```bash
   $ sudo ip -c -br link 
```

Для Windows:

```commandline
  > netsh interface show interface
  > getmac /V
```

2. #### Какой протокол используется для распознавания соседа по сетевому интерфейсу? Какой пакет и команды есть в Linux для этого?

Link Layer Discovery Protocol (LLDP) — протокол канального уровня

```bash
   $ sudo apt install lldpd
   $ sudo systemctl start lldpd.service
   $ sudo lldpctl
-------------------------------------------------------------------------------
LLDP neighbors:
-------------------------------------------------------------------------------
```

3. #### Какая технология используется для разделения L2 коммутатора на несколько виртуальных сетей? Какой пакет и команды есть в Linux для этого? Приведите пример конфига.

Для разделения коммутатора на виртуальные сети на канальном уровне применяется технология VLAN. Для операционной системы Linux пакет `vlan`

```bash
   $ sudo apt install vlan 
   $ cat /etc/network/interfaces.d/eth0.4000 
auto eth0.4000
iface eth0.4000 inet static
        address 172.17.10.1
        netmask 255.255.255.0
        vlan_raw_device eth0
```

4. #### Какие типы агрегации интерфейсов есть в Linux? Какие опции есть для балансировки нагрузки? Приведите пример конфига.

```bash
mode=0 (balance-rr)
mode=1 (active-backup)
mode=2 (balance-xor)
mode=3 (broadcast)
mode=4 (802.3ad)
mode=5 (balance-tlb)
mode=6 (balance-alb)
  $ cat /etc/network/interfaces.d/bond0
auto bond0
iface bond0 inet static
    address 192.168.137.5
    netmask 255.255.255.0
    network 192.168.137.0
    gateway 192.168.137.1
    bond-slaves eth1 eth2
    bond-mode active-backup
    bond-miimon 100
    bond-downdelay 200
    bond-updelay 200
```

5. #### Сколько IP адресов в сети с маской /29 ? Сколько /29 подсетей можно получить из сети с маской /24. Приведите несколько примеров /29 подсетей внутри сети 10.10.10.0/24.

```bash
  $ ipcalc -b 0.0.0.0/29 | grep Hosts
Hosts/Net: 6                     Class A
  В подсети /29 6 адресов;
  $ calc 256/8
        32
  Из сети с маской /24 можно получить 32 подсети с маской /29. 
  $ ipcalc -b 10.10.10.0/24 -s 6 6 6
Address:   10.10.10.0
Netmask:   255.255.255.0 = 24
Wildcard:  0.0.0.255
=>
Network:   10.10.10.0/24
HostMin:   10.10.10.1
HostMax:   10.10.10.254
Broadcast: 10.10.10.255
Hosts/Net: 254                   Class A, Private Internet

1. Requested size: 6 hosts
Netmask:   255.255.255.248 = 29
Network:   10.10.10.0/29
HostMin:   10.10.10.1
HostMax:   10.10.10.6
Broadcast: 10.10.10.7
Hosts/Net: 6                     Class A, Private Internet

2. Requested size: 6 hosts
Netmask:   255.255.255.248 = 29
Network:   10.10.10.8/29
HostMin:   10.10.10.9
HostMax:   10.10.10.14
Broadcast: 10.10.10.15
Hosts/Net: 6                     Class A, Private Internet

3. Requested size: 6 hosts
Netmask:   255.255.255.248 = 29
Network:   10.10.10.16/29
HostMin:   10.10.10.17
HostMax:   10.10.10.22
Broadcast: 10.10.10.23
Hosts/Net: 6                     Class A, Private Internet

Needed size:  24 addresses.
Used network: 10.10.10.0/27
Unused:
10.10.10.24/29
10.10.10.32/27
10.10.10.64/26
10.10.10.128/25
```

6. #### Задача: вас попросили организовать стык между 2-мя организациями. Диапазоны 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 уже заняты. Из какой подсети допустимо взять частные IP адреса? Маску выберите из расчета максимум 40-50 хостов внутри подсети.

```bash
   $ ipcalc -b 100.64.0.0/26
Address:   100.64.0.0
Netmask:   255.255.255.192 = 26
Wildcard:  0.0.0.63
=>
Network:   100.64.0.0/26
HostMin:   100.64.0.1
HostMax:   100.64.0.62
Broadcast: 100.64.0.63
Hosts/Net: 62                    Class A
```

7. #### Как проверить ARP таблицу в Linux, Windows? Как очистить ARP кеш полностью? Как из ARP таблицы удалить только один нужный IP?

Для Linux:

```bash
   $ ip neighbor
   $ sudo ip neigh flush all
   $ ip neighbor del 172.26.96.1
```

Для Windows:

```commandline
   > arp -a
   > arp -d 192.168.10.5
   > arp -d *
```
---
