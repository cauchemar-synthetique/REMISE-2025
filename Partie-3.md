# Part III : Storage is still disks in 2025

> Il a beaucoup beaucoup trop de features de fou, il se contente pas de couper des disques !

🌞 **Afficher l'état actuel de LVM**

- afficher la liste des *PV* (*Volume Volumes*)
  - ce sont les disque durs et partitions physiques que LVM gère

```ps

[cauchemar@node1 ~]$ sudo su
[root@node1 cauchemar]# pvs
  PV             VG Fmt  Attr PSize  PFree
  /dev/nvme0n1p1 rl lvm2 a--  15.00g 4.00m
[root@node1 cauchemar]# vgs
  VG #PV #LV #SN Attr   VSize  VFree
  rl   1   2   0 wz--n- 15.00g 4.00m
[root@node1 cauchemar]# lvs
  LV   VG Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  home rl -wi-ao----  5.00g
  root rl -wi-ao---- 10.00g
[root@node1 cauchemar]#

```

🌞 **Déterminer le type de système de fichiers**

- de la partition montée sur `/`
- de la partition montée sur `/home`
- **attention** : 
  - j'attends une commande qui détecte le type de système de fichiers sur une partition donnée : `<COMMANDE> /dev/chemin/partition`
  - je ne VEUX PAS une commande qui affiche les partitions actuellement utilisées où on voit le système de fichiers utilisé (pas de `mount` par exemple)
```ps
[root@node1 cauchemar]# exit
exit
[cauchemar@node1 ~]$
[cauchemar@node1 ~]$
[cauchemar@node1 ~]$ lsblk
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
[cauchemar@node1 ~]$ blkid /dev/mapper/rl-root
/dev/mapper/rl-root: UUID="5c90af69-1b9f-4437-986f-472c7d9a32f5" TYPE="ext4"
[cauchemar@node1 ~]$ blkid /dev/mapper/rl-home
/dev/mapper/rl-home: UUID="6b75af51-8662-4a11-a4c3-6c9fd0edb80b" TYPE="ext4"
[cauchemar@node1 ~]$

```
## 2. HELP my partition is full


🌞 **Remplissez votre partition `/home`**

- on va simuler avec un truc bourrin :

```
dd if=/dev/zero of=/home/<TON_USER>/bigfile bs=4M count=2500
```

> 2500x4M ça fait 20G. Ca fait trop.

🌞 **Constater que la partition est pleine**

- avec un `df -h`

🌞 **Agrandir la partition**

- avec des commandes LVM il faut agrandir le logical volume
- ensuite il faudra indiquer au système de fichier ext4 que la partition a été agrandie
- prouvez avec un `df -h` que vous avez récupéré de l'espace en plus

🌞 **Remplissez votre partition `/home`**

- on va simuler encore avec un truc bourrin :

```
dd if=/dev/zero of=/home/<TON_USER>/bigfile bs=4M count=2500
```

> 2500x4M ça fait toujours 20G. Et ça fait toujours trop.

➜ **Eteignez la VM et ajoutez lui un disque de 40G**

🌞 **Utiliser ce nouveau disque pour étendre la partition `/home` de 20G**

- dans l'ordre il faut :
- indiquer à LVM qu'il y a un nouveau PV dispo
- ajouter ce nouveau PV au VG existant
- étendre le LV existant pour récupérer le nouvel espace dispo au sein du VG
- indiquer au système de fichier ext4 que la partition a été agrandie
- prouvez avec un `df -h` que vous avez récupéré de l'espace en plus

## 3. Prepare another partition

Pour la suite du TP, on va préparer une dernière partition. Il devrait vous rester 20G de libre avec le disque de 40 que vous venez d'ajouter.

**Cette partition contiendra des fichiers HTML pour des sites web (fictifs).**

🌞 **Créez une nouvelle partition**

- le LV doit s'appeler `web`
- elle doit faire 20G et être formatée en ext4
- il faut la monter sur `/var/www`

🌞 **Proposez au moins une option de montage**

- au moment où on monte la partition (avec fstab ou la commande `mount`), on peut choisir des options de montage
- proposez au moins une option de montage qui augmente le niveau de sécurité lors de l'utilisation de la partition
- je rappelle que la partition ne contiendra que des fichiers HTML
