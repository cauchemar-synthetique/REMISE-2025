# Part I : Rocky install

🌞 **Prouvez que le schéma de partitionnement a bien été appliqué**

PS C:\WINDOWS\system32> ssh cauchemar@10.1.1.11
cauchemar@10.1.1.11's password:
Last login: Mon Feb 17 14:42:08 2025
[cauchemar@localhost ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sr0          11:0    1  1.8G  0 rom
nvme0n1     259:0    0   50G  0 disk
├─nvme0n1p1 259:1    0   15G  0 part
│ ├─rl-root 253:0    0   10G  0 lvm  /
│ └─rl-home 253:1    0    5G  0 lvm  /home
├─nvme0n1p2 259:2    0    5G  0 part /var
├─nvme0n1p3 259:3    0    4G  0 part [SWAP]
├─nvme0n1p4 259:4    0    1K  0 part
└─nvme0n1p5 259:5    0  512M  0 part /boot
[cauchemar@localhost ~]$
[cauchemar@localhost ~]$ df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                1.8G     0  1.8G   0% /dev/shm
tmpfs                726M  9.0M  717M   2% /run
/dev/mapper/rl-root  9.8G  1.3G  8.0G  14% /
/dev/nvme0n1p5       488M  273M  179M  61% /boot
/dev/nvme0n1p2       4.9G  181M  4.4G   4% /var
/dev/mapper/rl-home  4.9G   44K  4.6G   1% /home
tmpfs                363M     0  363M   0% /run/user/1000
[cauchemar@localhost ~]$

➜ Par défaut, sous Rocky Linux :

- il existe un groupe appelé `wheel` déjà créé à l'installation
- le groupe `wheel` est déjà dans la conf `sudo` pour autoriser ses membres à utiliser les droits de `root` avec la commande `sudo`

🌞 **Mettre en évidence la ligne de configuration `sudo` qui concerne le groupe `wheel`**

- avec un `cat TRUC | grep TRUC` je veux voir que la bonne ligne

[cauchemar@localhost ~]$ sudo cat /etc/sudoers | grep -E '^[^#]*wheel'
[sudo] password for cauchemar:
%wheel  ALL=(ALL)       ALL
[cauchemar@localhost ~]$

🌞 **Prouvez que votre utilisateur est bien dans le groupe `wheel`**

[cauchemar@localhost ~]$ groups
cauchemar wheel
[cauchemar@localhost ~]$

🌞 **Prouvez que la langue configurée pour l'OS est bien l'anglais**

[cauchemar@localhost ~]$ echo $LANG
C.UTF-8
[cauchemar@localhost ~]$

- je veux une ligne de commande qui affiche la langue actuelle de l'OS
- que vos messages d'erreur soient en anglais ça me suffit pas ;D

🌞 **Prouvez que le firewall est déjà actif**

[cauchemar@localhost ~]$ sudo systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-02-17 14:41:19 CET; 15min ago
       Docs: man:firewalld(1)
   Main PID: 727 (firewalld)
      Tasks: 2 (limit: 22949)
     Memory: 44.9M
        CPU: 498ms
     CGroup: /system.slice/firewalld.service
             └─727 /usr/bin/python3 -s /usr/sbin/firewalld --nofork --nopid

Feb 17 14:41:18 localhost systemd[1]: Starting firewalld - dynamic firewall daemon...
Feb 17 14:41:19 localhost systemd[1]: Started firewalld - dynamic firewall daemon.
[cauchemar@localhost ~]$

- le service de firewalling s'appelle `firewalld` sous Rocky (on le manipule avec la commande `firewall-cmd`)
