# Part V : OpenSSH Server

**Le serveur OpenSSH est strictement nécessaire à l'administration, et occupe aussi une place cruciale dans le niveau de sécurité d'une machine.**

En effet, on parle d'un programme qui tourne en `root` (obligé...), qui écoute sur un port réseau (il est donc attaquable, c'est une porte potentiellement ouverte), et qui en plus, bah sert à prendre le contrôle d'une machine à distance.

Besoin d'un dessin pour expliquer à quel point c'est sensible ?

Néanmoins nécessaire partout.

## Index

- [Part V : OpenSSH Server](#part-v--openssh-server)
  - [Index](#index)
  - [1. Basics](#1-basics)
  - [2. Authentication modes](#2-authentication-modes)
    - [A. Key-based authentication](#a-key-based-authentication)
  - [3. Bonus : Cert-based authentication](#3-bonus--cert-based-authentication)
  - [4. Further hardening](#4-further-hardening)
  - [5. fail2ban](#5-fail2ban)
  - [6. Automatisation](#6-automatisation)

## 1. Basics

🌞 **Afficher l'identifiant du processus serveur OpenSSH en cours d'exécution**

```ps
[cauchemar@node1 ~]$ ps aux | grep "[s]shd"
root        1327  0.0  0.5  16792  9344 ?        Ss   11:10   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
```

> On peut aussi obtenir l'info avec un `systemctl status` 

```ps
[cauchemar@node1 ~]$ systemctl status sshd | grep "Main PID"
   Main PID: 1327 (sshd)
```

🌞 **Changer le port d'écoute du serveur OpenSSH**

- prouvez que votre changement a pris effet
```ps
[cauchemar@node1 ~]$ sudo ss -tulnp | grep 33000
tcp   LISTEN 0      128        10.1.1.11:33000      0.0.0.0:*    users:(("sshd",pid=1527,fd=3))
```

- prouvez que vous pouvez toujours vous connecter à la machine en SSH, sur ce nouveau port
```ps
PS C:\Users\cauchemar> ssh -p 33000 cauchemar@10.1.1.11
cauchemar@10.1.1.11 s password:
Last login: Thu Feb 20 14:25:01 2025
[cauchemar@node1 ~]$
```
- expliquez pourquoi on considère parfois utile de changer le port d'écoute par défaut du serveur SSH

Evite les scans de bots ou tout autres attaques qui visent le port 22 par défaut

## 2. Authentication modes

### A. Key-based authentication

🌞 **Configurer une authentification par clé**

```ps
PS C:\Users\cauchemar> ssh-keygen -t rsa -b 4096 -f C:\Users\cauchemar\.ssh\id_rsa

PS C:\Users\cauchemar> cat ~/.ssh/id_rsa.pub | ssh -p 33000 cauchemar@10.1.1.11 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

PS C:\Users\cauchemar> ssh -p 33000 cauchemar@10.1.1.11
Last login: Thu Feb 20 15:33:11 2025 from 10.1.1.0
```

🌞 **Désactiver la connexion par password**
```ps
[cauchemar@node1 ~]$ sudo grep PasswordAuthentication /etc/ssh/sshd_config
PasswordAuthentication no
```

🌞 **Désactiver la connexion en tant que `root`**
```ps
[cauchemar@node1 ~]$ sudo grep PermitRootLogin /etc/ssh/sshd_config
PermitRootLogin no
```
```ps
PS C:\Users\cauchemar> ssh -p 33000 root@10.1.1.11
root@10.1.1.11: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

## 3. Further hardening

```ps
MaxAuthTries 3
MaxSessions 3
AllowGroups sshusers
PermitEmptyPasswords no
HostbasedAuthentication no
```

## 5. fail2ban

> Un outil extrêmement récurrent dans le monde Linux : un premier rempart contre les attaques de bruteforce.

🌞 **Installer fail2ban sur la machine**

🌞 **Configurer fail2ban**

```ps
[cauchemar@node1 ~]$ sudo nano /etc/fail2ban/jail.local
[sshd]
port    = ssh
logpath = %(sshd_log)s
backend = %(sshd_backend)s
enabled = true
filter = sshd
maxretry = 7
findtime = 300
bantime = 600
```
```ps
[cauchemar@node1 ~]$ sudo systemctl restart fail2ban
```

🌞 **Prouvez que fail2ban est effectif**

```ps
[cauchemar@node1 ~]$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 1
|  |- Total failed:     8
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd + _COMM=sshd-session
`- Actions
   |- Currently banned: 1
   |- Total banned:     1
   `- Banned IP list:   10.1.1.0
```
- levez le ban avec une commande adaptée

```ps
[cauchemar@node1 ~]$ sudo fail2ban-client set sshd unbanip 10.1.1.0
1

[cauchemar@node1 ~]$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 1
|  |- Total failed:     8
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd + _COMM=sshd-session
`- Actions
   |- Currently banned: 0
   |- Total banned:     1
   `- Banned IP list:
```

## 6. Automatisation

Dernière section : un peu de dév en bash pour automatiser toute la configuration que vous venez de faire.

L'idée est simple : écrire un script shell qui applique la configuration de cette Partie V (openSSH et fail2ban) sur une machine Rocky Linux fraîchement installée.

🌞 **Ecrire le script `harden.sh`**

Voilà le fichier -> [harden.sh](./harden.sh)
 
 ```ps
 [cauchemar@node1 ~]$ sudo ./harden.sh
active
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     8
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd + _COMM=sshd-session
`- Actions
   |- Currently banned: 0
   |- Total banned:     1
   `- Banned IP list:
```

