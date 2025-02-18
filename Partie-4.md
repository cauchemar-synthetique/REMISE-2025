# Part IV : User management

**Hum, cette partie est censée être envoyée vite fait bien fait ! Prouvez-le moi :D**

Gestion d'utilisateurs, de mot de passe, et de `sudo` ! Puis dans un deuxième temps, on continue sur la gestion de permissions.

## Index

- [Part IV : User management](#part-iv--user-management)
  - [Index](#index)
  - [1. Users](#1-users)
    - [A. Master what already exists](#a-master-what-already-exists)
    - [B. User creation and configuration](#b-user-creation-and-configuration)
    - [C. Hackers gonna hack](#c-hackers-gonna-hack)
  - [2. Files and permissions](#2-files-and-permissions)
    - [A. Listing POSIX permissions](#a-listing-posix-permissions)
    - [B. Protect a file using permissions](#b-protect-a-file-using-permissions)
    - [C. Extended attributes](#c-extended-attributes)

## 1. Users

### A. Master what already exists

🌞 **Déterminer l'existant :**

- lister tous les utilisateurs créés sur la machine
```ps
[cauchemar@node1 ~]$ getent passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin
tss:x:59:59:Account used for TPM access:/:/usr/sbin/nologin
systemd-coredump:x:999:997:systemd Core Dumper:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
sssd:x:998:996:User for sssd:/:/sbin/nologin
chrony:x:997:995:chrony system user:/var/lib/chrony:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
cauchemar:x:1000:1000:cauchemar:/home/cauchemar:/bin/bash
tcpdump:x:72:72::/:/sbin/nologin
nginx:x:996:993:Nginx web server:/var/lib/nginx:/sbin/nologin
[cauchemar@node1 ~]$
```
- lister tous les groupes d'utilisateur
```ps

[cauchemar@node1 ~]$ getent group
root:x:0:
bin:x:1:
daemon:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
mem:x:8:
kmem:x:9:
wheel:x:10:cauchemar
cdrom:x:11:
mail:x:12:
man:x:15:
dialout:x:18:
floppy:x:19:
games:x:20:
tape:x:33:
video:x:39:
ftp:x:50:
lock:x:54:
audio:x:63:
users:x:100:
nobody:x:65534:
utmp:x:22:
utempter:x:35:
ssh_keys:x:101:
tss:x:59:
input:x:999:
kvm:x:36:
render:x:998:
systemd-journal:x:190:
systemd-coredump:x:997:
dbus:x:81:
sssd:x:996:
chrony:x:995:
sshd:x:74:
sgx:x:994:
cauchemar:x:1000:
tcpdump:x:72:
nginx:x:993:
[cauchemar@node1 ~]$

```

- déterminer la liste des groupes dans lesquels se trouvent votre utilisateur

```ps
[cauchemar@node1 ~]$ groups cauchemar
cauchemar : cauchemar wheel
[cauchemar@node1 ~]$

```

🌞 **Lister tous les processus qui sont actuellement en cours d'exécution, lancés par `root`**

```ps
[cauchemar@node1 ~]$ ps -U root -u root
    PID TTY          TIME CMD
      1 ?        00:00:02 systemd
      2 ?        00:00:00 kthreadd
      3 ?        00:00:00 pool_workqueue_
      4 ?        00:00:00 kworker/R-rcu_g
      5 ?        00:00:00 kworker/R-rcu_p
      6 ?        00:00:00 kworker/R-slub_
      7 ?        00:00:00 kworker/R-netns
      9 ?        00:00:00 kworker/0:0H-events_highpri
     10 ?        00:00:00 kworker/u512:0-events_unbound
     11 ?        00:00:00 kworker/R-mm_pe
     12 ?        00:00:00 kworker/u512:1-netns
     13 ?        00:00:00 rcu_tasks_kthre
     14 ?        00:00:00 rcu_tasks_rude_
     15 ?        00:00:00 rcu_tasks_trace
     16 ?        00:00:00 ksoftirqd/0
     17 ?        00:00:00 rcu_preempt
     18 ?        00:00:00 migration/0
     19 ?        00:00:00 idle_inject/0
     21 ?        00:00:00 cpuhp/0
     22 ?        00:00:00 cpuhp/1
     23 ?        00:00:00 idle_inject/1
     24 ?        00:00:00 migration/1
     25 ?        00:00:00 ksoftirqd/1
     27 ?        00:00:00 kworker/1:0H-events_highpri
     29 ?        00:00:00 kworker/u514:0-events_unbound
     32 ?        00:00:00 kdevtmpfs
     33 ?        00:00:00 kworker/R-inet_
     34 ?        00:00:00 kworker/u513:1-events_unbound
     35 ?        00:00:00 kauditd
     37 ?        00:00:00 khungtaskd
     38 ?        00:00:00 oom_reaper
     39 ?        00:00:00 kworker/R-write
     40 ?        00:00:00 kcompactd0
     41 ?        00:00:00 ksmd
     44 ?        00:00:00 khugepaged
     45 ?        00:00:00 kworker/R-crypt
     46 ?        00:00:00 kworker/R-kinte
     47 ?        00:00:00 kworker/R-kbloc
     48 ?        00:00:00 kworker/R-blkcg
     49 ?        00:00:00 kworker/R-tpm_d
     50 ?        00:00:00 kworker/R-md
     51 ?        00:00:00 kworker/R-md_bi
     52 ?        00:00:00 kworker/R-edac-
     53 ?        00:00:00 watchdogd
     54 ?        00:00:00 kworker/1:1H-kblockd
     55 ?        00:00:00 kswapd0
     60 ?        00:00:00 kworker/R-kthro
     66 ?        00:00:00 irq/24-pciehp
     67 ?        00:00:00 irq/25-pciehp
     68 ?        00:00:00 irq/26-pciehp
     69 ?        00:00:00 irq/27-pciehp
     70 ?        00:00:00 irq/28-pciehp
     71 ?        00:00:00 irq/29-pciehp
     72 ?        00:00:00 irq/30-pciehp
     73 ?        00:00:00 irq/31-pciehp
     74 ?        00:00:00 irq/32-pciehp
     75 ?        00:00:00 irq/33-pciehp
     76 ?        00:00:00 irq/34-pciehp
     77 ?        00:00:00 irq/35-pciehp
     78 ?        00:00:00 irq/36-pciehp
     79 ?        00:00:00 irq/37-pciehp
     80 ?        00:00:00 irq/38-pciehp
     81 ?        00:00:00 irq/39-pciehp
     82 ?        00:00:00 irq/40-pciehp
     83 ?        00:00:00 irq/41-pciehp
     84 ?        00:00:00 irq/42-pciehp
     85 ?        00:00:00 irq/43-pciehp
     86 ?        00:00:00 irq/44-pciehp
     87 ?        00:00:00 irq/45-pciehp
     88 ?        00:00:00 irq/46-pciehp
     89 ?        00:00:00 irq/47-pciehp
     90 ?        00:00:00 irq/48-pciehp
     91 ?        00:00:00 irq/49-pciehp
     92 ?        00:00:00 irq/50-pciehp
     93 ?        00:00:00 irq/51-pciehp
     94 ?        00:00:00 irq/52-pciehp
     95 ?        00:00:00 irq/53-pciehp
     96 ?        00:00:00 irq/54-pciehp
     97 ?        00:00:00 irq/55-pciehp
     98 ?        00:00:00 kworker/R-acpi_
     99 ?        00:00:00 kworker/R-kmpat
    100 ?        00:00:00 kworker/R-kalua
    101 ?        00:00:00 kworker/R-mld
    102 ?        00:00:00 kworker/R-ipv6_
    112 ?        00:00:00 kworker/R-kstrp
    223 ?        00:00:00 kworker/u515:0
    224 ?        00:00:00 kworker/u516:0
    225 ?        00:00:00 kworker/u517:0
    263 ?        00:00:00 kworker/0:1H-kblockd
    423 ?        00:00:00 kworker/R-ata_s
    427 ?        00:00:00 kworker/R-nvme-
    428 ?        00:00:00 scsi_eh_0
    430 ?        00:00:00 kworker/R-nvme-
    431 ?        00:00:00 kworker/R-scsi_
    433 ?        00:00:00 kworker/R-nvme-
    434 ?        00:00:00 kworker/R-nvme-
    437 ?        00:00:00 scsi_eh_1
    438 ?        00:00:00 kworker/R-scsi_
    533 ?        00:00:00 kworker/R-kdmfl
    551 ?        00:00:00 jbd2/dm-0-8
    552 ?        00:00:00 kworker/R-ext4-
    615 ?        00:00:00 systemd-journal
    628 ?        00:00:00 systemd-udevd
    665 ?        00:00:00 jbd2/nvme0n1p2-
    669 ?        00:00:00 kworker/R-ext4-
    674 ?        00:00:00 jbd2/nvme0n1p5-
    675 ?        00:00:00 kworker/R-ext4-
    677 ?        00:00:00 kworker/R-kdmfl
    681 ?        00:00:00 irq/75-vmw_vmci
    682 ?        00:00:00 irq/76-vmw_vmci
    683 ?        00:00:00 irq/77-vmw_vmci
    691 ?        00:00:00 irq/16-vmwgfx
    692 ?        00:00:00 kworker/R-ttm
    708 ?        00:00:00 jbd2/dm-1-8
    709 ?        00:00:00 kworker/R-ext4-
    721 ?        00:00:00 auditd
    751 ?        00:00:00 firewalld
    752 ?        00:00:00 irqbalance
    753 ?        00:00:00 systemd-logind
    760 ?        00:00:00 NetworkManager
    806 ?        00:00:00 crond
    807 ?        00:00:00 login
    845 ?        00:00:00 rsyslogd
   1294 ?        00:00:00 sshd
   1333 ?        00:00:00 sshd
   1488 ?        00:00:00 kworker/R-kdmfl
   1521 ?        00:00:00 sshd
   1552 ?        00:00:00 kworker/u514:3-flush-253:0
   1620 ?        00:00:00 jbd2/dm-2-8
   1621 ?        00:00:00 kworker/R-ext4-
   1624 ?        00:00:00 kworker/u513:0-events_unbound
   1639 ?        00:00:14 kworker/1:1-rcu_gp
   1656 ?        00:00:00 sshd
   1683 ?        00:00:06 kworker/1:0-events
   1745 ?        00:00:00 kworker/R-tls-s
   1885 ?        00:00:00 kworker/u514:1-events_unbound
   1895 ?        00:00:00 nginx
   1914 ?        00:00:00 kworker/0:1-mm_percpu_wq
   1916 ?        00:00:00 kworker/0:0-mm_percpu_wq
   1919 ?        00:00:00 kworker/0:2-ata_sff
```

🌞 **Lister tous les processus qui sont actuellement en cours d'exécution, lancés par votre utilisateur**

```ps
[cauchemar@node1 ~]$ ps -U cauchemar -u cauchemar
    PID TTY          TIME CMD
   1299 ?        00:00:00 systemd
   1301 ?        00:00:00 (sd-pam)
   1308 tty1     00:00:00 bash
   1338 ?        00:00:00 sshd
   1339 pts/0    00:00:00 bash
   1525 ?        00:00:00 sshd
   1526 pts/1    00:00:00 bash
   1660 ?        00:00:00 sshd
   1661 pts/2    00:00:00 bash
   1925 pts/2    00:00:00 ps
[cauchemar@node1 ~]$

```

🌞 **Déterminer le hash du mot de passe de `root`**

```ps
[cauchemar@node1 ~]$ sudo grep '^root:' /etc/shadow
[sudo] password for cauchemar:
root:$6$a1ADAHwhtDoM5pgg$er7TFDSsklmipzCfqaBUzoh2e5aSt/VuVhDLTB5w05eC7IjSP3/zCYOySDDahvBYnDdomJJAFM/CGZaUz8eTn.::0:99999:7:::
[cauchemar@node1 ~]$
```

🌞 **Déterminer le hash du mot de passe de votre utilisateur**

```ps
[cauchemar@node1 ~]$ sudo grep '^cauchemar:' /etc/shadow
cauchemar:$6$m3Vyv.x9XNWsiReo$NmQl9QL43vxYaI0W8p6xgI.hhjydcT9Y0Ki2v3h.464VZx.Q5oRGNfrOVIoB63IHiw.mNR3Pg5baLxHvaWLbj/::0:99999:7:::
[cauchemar@node1 ~]$
```

🌞 **Déterminer la fonction de hachage qui a été utilisée**

```ps
[cauchemar@node1 ~]$ sudo grep '^cauchemar:' /etc/shadow | cut -d":" -f2 | cut -d"$" -f4
NmQl9QL43vxYaI0W8p6xgI.hhjydcT9Y0Ki2v3h.464VZx.Q5oRGNfrOVIoB63IHiw.mNR3Pg5baLxHvaWLbj/
[cauchemar@node1 ~]$
```

🌞 **Déterminer, pour l'utilisateur `root`** :

```ps
[cauchemar@node1 ~]$ sudo grep '^root:' /etc/shadow | cut -d":" -f2 | cut -d"$" -f4
[sudo] password for cauchemar:
er7TFDSsklmipzCfqaBUzoh2e5aSt/VuVhDLTB5w05eC7IjSP3/zCYOySDDahvBYnDdomJJAFM/CGZaUz8eTn.
[cauchemar@node1 ~]$

```

🌞 **Afficher la ligne de configuration du fichier `sudoers` qui permet à votre utilisateur d'utiliser `sudo`**

![sudo](./img/sudo.png)

```ps
[cauchemar@node1 ~]$ sudo grep -r 'ALL=(ALL' /etc/sudoers*
/etc/sudoers:root       ALL=(ALL)       ALL
/etc/sudoers:%wheel     ALL=(ALL)       ALL
/etc/sudoers:# %wheel   ALL=(ALL)       NOPASSWD: ALL
[cauchemar@node1 ~]$

```

### B. User creation and configuration

🌞 **Créer un utilisateur :**

- doit s'appeler `meow`
- ne doit appartenir QUE à un groupe nommé `admins`
- ne doit pas avoir de répertoire personnel utilisable
- ne doit pas avoir un shell utilisable

> Il s'agit donc ici d'un utilisateur avec lequel on pourra pas se connecter à la machine (ni en console, ni en SSH).

🌞 **Configuration `sudoers`**

- ajouter une configuration `sudoers` pour que l'utilisateur `meow` puisse exécuter seulement et uniquement les commandes `ls`, `cat`, `less` et `more` en tant que votre utilisateur
- ajouter une configuration `sudoers` pour que les membres du groupe `admins` puisse exécuter seulement et uniquement la commande `apt` en tant que `root`
- ajouter une configuration `sudoers` pour que votre utilisateur puisse exécuter n'importe quel commande en tant `root`, sans avoir besoin de saisir un mot de passe
- prouvez que ces 3 configurations ont pris effet (vous devez vous authentifier avec le bon utilisateur, et faire une commande `sudo` qui doit fonctioner correctement)

> Pour chaque point précédent, c'est une seule ligne de configuration à ajouter dans le fichier `sudoers` de la machine.

### C. Hackers gonna hack

🌞 **Déjà une configuration faible ?**

- l'utilisateur `meow` est en réalité complètement `root` sur la machine hein là. Prouvez-le.
- proposez une configuration similaire, sans présenter cette faiblesse de configuration
  - vous pouvez ajouter de la configuration
  - ou supprimer de la configuration
  - du moment qu'on garde des fonctionnalités à peu près équivalentes !

## 2. Files and permissions

**Dans un OS, en particulier Linux, on dit souvent que "tout est fichier".**

En effet, que ce soit les programmes (que ce soit `ls`, ou Firefox, ou Steam, ou le kernel), les fichiers personnels, les fichiers de configuration, et bien d'autres, **l'ensemble des composants d'un OS, et tout ce qu'on peut y ajouter se résume à un gros tas de fichiers.**

Gérer correctement les permissions des fichiers est une étape essentielle dans le renforcement d'une machine.

**C'est la première barrière de sécurité, (beaucoup) trop souvent négligée, alors qu'elle est extrêmement efficace et robuste.**

### A. Listing POSIX permissions

🌞 **Déterminer les permissions des fichiers/dossiers...**

- le fichier qui contient la liste des utilisateurs
- le fichier qui contient la liste des hashes des mots de passe des utilisateurs
- le fichier de configuration du serveur OpenSSH
- le répertoire personnel de l'utilisateur `root`
- le répertoire personnel de votre utilisateur
- le programme `ls`
- le programme `systemctl`

> POSIX c'est le nom d'un standard qui regroupe plein de concepts avec lesquels vous êtes finalement déjà familiers. Les permissions rwx qu'on retrouve sous les OS Linux (et MacOS, et BSD, et d'autres) font partie de ce standard et sont donc appelées "permissions POSIX".

![Windows POSIX](./img/posix_compliant.png)

### B. Protect a file using permissions

🌞 **Restreindre l'accès à un fichier personnel**

- créer un fichier nommé `dont_readme.txt` (avec le contenu de votre choix)
- il doit se trouver dans un dossier lisible et écrivable par tout le monde
- faites en sorte que seul votre utilisateur (pas votre groupe) puisse lire ou modifier ce fichier
- personne ne doit pouvoir l'exécuter
- prouvez que :
  - votre utilisateur peut le lire
  - votre utilisateur peut le modifier
  - l'utilisateur `meow` ne peut pas y toucher
  - l'utilisateur `root` peut quand même y toucher

> C'est l'un des "superpouvoirs" de `root` : contourner les permissions POSIX (les permissions `rwx`). On verra bien assez tôt que `root` n'a pas de "superpouvoirs" mais que ces contournements sont liés à une mécanique qu'on appelle les *capabilites*. C'est pour plus tard ! :)

### C. Extended attributes

🌞 **Lister tous les programmes qui ont le bit SUID activé**

🌞 **Rendre le fichier `dont_readme.txt` immuable**

- ça se fait avec les attributs étendus
- "immuable" ça veut dire qu'il ne peut plus être modifié DU TOUT : il est donc en read-only
- prouvez que le fichier ne peut plus être modifié par **personne**
