# Part II : Networking

Le réseau c'est la porte d'entrée pour toutes les autres machines. C'est le seul moyen d'être attaqué à distance.

Maîtriser au mieux le réseau d'une machine est donc primordial pour prétendre en renforcer la sécurité.

## Index

- [Part II : Networking](#part-ii--networking)
  - [Index](#index)
  - [1. Basic networking conf](#1-basic-networking-conf)
    - [A. Static IP](#a-static-ip)
    - [B. Hostname](#b-hostname)
  - [2. Listening ports](#2-listening-ports)
  - [3. Firewalling](#3-firewalling)

## 1. Basic networking conf

### A. Static IP

🌞 **Attribuer l'adresse IP `10.1.1.11/24`** à la VM

- ça veut dire que votre PC a pour adresse IP `10.1.1.X/24` (il est dans le même réseau)
- je vous file les instructions pour la définition de l'IP dans la VM, avec Rocky Linux on peut faire comme ça pour la définition d'une IP statique :

```ps 
DEVICE=ens160
NAME=lan

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.1.1.11
NETMASK=255.255.255.0

[cauchemar@localhost network-scripts]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:1c:69:15 brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    inet 10.1.1.11/24 brd 10.1.1.255 scope global noprefixroute ens160
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe1c:6915/64 scope link
       valid_lft forever preferred_lft forever
[cauchemar@localhost network-scripts]$
```

### B. Hostname

🌞 **Attribuer le nom `node1.tp1.b3` à la VM**

- ça se fait avec une commande `hostnamectl` en 2025 svp
- 
```ps
sudo hostnamectl set-hostname node1.tp1.b3

[cauchemar@localhost network-scripts]$ hostname
node1.tp1.b3
[cauchemar@localhost network-scripts]$

```

## 2. Listening ports

🌞 **Déterminer la liste des programmes qui écoutent sur un port TCP**

```ps
[cauchemar@localhost network-scripts]$ ss -tlen
State   Recv-Q  Send-Q   Local Address:Port   Peer Address:Port  Process
LISTEN  0       128            0.0.0.0:22          0.0.0.0:*      ino:21256 sk:2 cgroup:/system.slice/sshd.service <->
LISTEN  0       128               [::]:22             [::]:*      ino:21258 sk:3 cgroup:/system.slice/sshd.service v6only:1 <->
[cauchemar@localhost network-scripts]$

```

🌞 **Déterminer la liste des programmes qui écoutent sur un port UDP**

```ps

[cauchemar@localhost network-scripts]$ ss -ulen
State  Recv-Q  Send-Q   Local Address:Port   Peer Address:Port Process
UNCONN 0       0            127.0.0.1:323         0.0.0.0:*     ino:20059 sk:4 cgroup:/system.slice/chronyd.service <->
UNCONN 0       0                [::1]:323            [::]:*     ino:20060 sk:5 cgroup:/system.slice/chronyd.service v6only:1 <->
[cauchemar@localhost network-scripts]$

```

## 3. Firewalling

![fw](./img/fw.png)

➜ **Vous pouvez afficher l'état actuel de `firewalld`, le firewall de Rocky Linux, avec :**

```ps

sudo firewall-cmd --list-all

```

🌞 **Pour chacun des ports précédemment repérés...**

- montrez qu'il existe une règle firewall qui autorise le trafic entrant sur ce port
- ou pas ?
  
```ps

[cauchemar@localhost network-scripts]$ sudo firewall-cmd --list-all
[sudo] password for cauchemar:
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[cauchemar@localhost network-scripts]$

```

> **Attention !** Le firewall de Rocky Linux, `firewalld`, a deux concepts pour ouvrir un port TCP/UDP. Soit on ouvre... un port avec `--add-port` et on le voit apparaître devant `ports:`. Soit on ouvre un "service" avec `--add-service` et on le voit apparaître devant `services:`. Chaque "service" est donc un port ouvert (et à fermer potentiellement à la question suivante ;) ).

🌞 **Fermez tous les ports inutilement ouverts dans le firewall**

- principe du moindre privilège encore et encore !
- pas besoin qu'un port soit ouvert si aucun service n'écoute dessus

  ```ps
  
[cauchemar@node1 ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources:
  services:
  ports: 22/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[cauchemar@node1 ~]$

```

🌞 **Pour toutes les applications qui sont en écoute sur TOUTES les adresses IP**

- dans Linux, ce sont les applications qui écoutent sur la pseudo-adresse IP `0.0.0.0` : ça signifie que toutes les adresses IP de la machine sont concernées
- modifier la configuration de l'application pour n'écouter que une seule IP : celle qui est nécessaire

```ps

[cauchemar@node1 ~]$ sudo systemctl restart sshd
[cauchemar@node1 ~]$ sudo ss -tulnp | grep '0.0.0.0'
udp   UNCONN 0      0          127.0.0.1:323       0.0.0.0:*    users:(("chronyd",pid=733,fd=5))
tcp   LISTEN 0      128        10.1.1.11:22        0.0.0.0:*    users:(("sshd",pid=1797,fd=3))
[cauchemar@node1 ~]$

```
