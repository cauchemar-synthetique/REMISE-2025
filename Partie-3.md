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

  ```ps

  [root@node1 cauchemar]# dd if=/dev/zero of=/home/cauchemar/bigfile bs=4M count=2500
dd: error writing '/home/cauchemar/bigfile': No space left on device
1235+0 records in
1234+0 records out
5179555840 bytes (5.2 GB, 4.8 GiB) copied, 2.97327 s, 1.7 GB/s
[root@node1 cauchemar]#

```

🌞 **Agrandir la partition**

```ps
[cauchemar@node1 ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sr0          11:0    1  1.8G  0 rom
nvme0n1     259:0    0   50G  0 disk
├─nvme0n1p1 259:1    0   15G  0 part
│ ├─rl-root 253:0    0   10G  0 lvm  /
│ └─rl-home 253:1    0 30.5G  0 lvm  /home
├─nvme0n1p2 259:2    0    5G  0 part /var
├─nvme0n1p3 259:3    0    4G  0 part [SWAP]
├─nvme0n1p4 259:4    0    1K  0 part
├─nvme0n1p5 259:5    0  512M  0 part /boot
└─nvme0n1p6 259:6    0 25.5G  0 part
  └─rl-home 253:1    0 30.5G  0 lvm  /home
```

> 2500x4M ça fait toujours 20G. Et ça fait toujours trop.

➜ **Eteignez la VM et ajoutez lui un disque de 40G**

🌞 **Utiliser ce nouveau disque pour étendre la partition `/home` de 20G**

```ps
[cauchemar@node1 ~]$ sudo vgextend rl /dev/nvme0n2
  Volume group "rl" successfully extended
[cauchemar@node1 ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sr0          11:0    1  1.8G  0 rom
nvme0n1     259:0    0   50G  0 disk
├─nvme0n1p1 259:1    0   15G  0 part
│ ├─rl-root 253:0    0   10G  0 lvm  /
│ └─rl-home 253:1    0 30.5G  0 lvm  /home
├─nvme0n1p2 259:2    0    5G  0 part /var
├─nvme0n1p3 259:3    0    4G  0 part [SWAP]
├─nvme0n1p4 259:4    0    1K  0 part
├─nvme0n1p5 259:5    0  512M  0 part /boot
└─nvme0n1p6 259:6    0 25.5G  0 part
  └─rl-home 253:1    0 30.5G  0 lvm  /home
nvme0n2     259:7    0   40G  0 disk
[cauchemar@node1 ~]$ sudo vgs
  VG #PV #LV #SN Attr   VSize  VFree
  rl   3   2   0 wz--n- 80.48g <40.00g
[cauchemar@node1 ~]$ sudo lvextend -L+20G /dev/mapper/rl-home
  Size of logical volume rl/home changed from <30.49 GiB (7805 extents) to <50.49 GiB (12925 extents).
  Logical volume rl/home successfully resized.
[cauchemar@node1 ~]$ sudo resize2fs /dev/mapper/rl-home
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/mapper/rl-home is mounted on /home; on-line resizing required
old_desc_blocks = 4, new_desc_blocks = 7
The filesystem on /dev/mapper/rl-home is now 13235200 (4k) blocks long.

[cauchemar@node1 ~]$ sudo vgs
  VG #PV #LV #SN Attr   VSize  VFree
  rl   3   2   0 wz--n- 80.48g <20.00g
[cauchemar@node1 ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sr0          11:0    1  1.8G  0 rom
nvme0n1     259:0    0   50G  0 disk
├─nvme0n1p1 259:1    0   15G  0 part
│ ├─rl-root 253:0    0   10G  0 lvm  /
│ └─rl-home 253:1    0 50.5G  0 lvm  /home
├─nvme0n1p2 259:2    0    5G  0 part /var
├─nvme0n1p3 259:3    0    4G  0 part [SWAP]
├─nvme0n1p4 259:4    0    1K  0 part
├─nvme0n1p5 259:5    0  512M  0 part /boot
└─nvme0n1p6 259:6    0 25.5G  0 part
  └─rl-home 253:1    0 50.5G  0 lvm  /home
nvme0n2     259:7    0   40G  0 disk
└─rl-home   253:1    0 50.5G  0 lvm  /home
[cauchemar@node1 ~]$ df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                1.8G     0  1.8G   0% /dev/shm
tmpfs                726M  9.1M  717M   2% /run
/dev/mapper/rl-root  9.8G  1.3G  8.0G  14% /
/dev/nvme0n1p2       4.9G  182M  4.4G   4% /var
/dev/nvme0n1p5       488M  273M  179M  61% /boot
/dev/mapper/rl-home   50G  4.9G   43G  11% /home
tmpfs                363M     0  363M   0% /run/user/1000
[cauchemar@node1 ~]$

```
## 3. Prepare another partition

Pour la suite du TP, on va préparer une dernière partition. Il devrait vous rester 20G de libre avec le disque de 40 que vous venez d'ajouter.

**Cette partition contiendra des fichiers HTML pour des sites web (fictifs).**

🌞 **Créez une nouvelle partition**

```ps
[cauchemar@node1 ~]$
[cauchemar@node1 ~]$
[cauchemar@node1 ~]$
[cauchemar@node1 ~]$ df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                1.8G     0  1.8G   0% /dev/shm
tmpfs                726M  9.1M  717M   2% /run
/dev/mapper/rl-root  9.8G  1.3G  8.0G  14% /
/dev/nvme0n1p2       4.9G  182M  4.4G   4% /var
/dev/nvme0n1p5       488M  273M  179M  61% /boot
/dev/mapper/rl-home   50G  4.9G   43G  11% /home
tmpfs                363M     0  363M   0% /run/user/1000
[cauchemar@node1 ~]$ sudo mount /dev/rl/web /var/www
[cauchemar@node1 ~]$ df h
df: h: No such file or directory
[cauchemar@node1 ~]$ df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                1.8G     0  1.8G   0% /dev/shm
tmpfs                726M  9.1M  717M   2% /run
/dev/mapper/rl-root  9.8G  1.3G  8.0G  14% /
/dev/nvme0n1p2       4.9G  182M  4.4G   4% /var
/dev/nvme0n1p5       488M  273M  179M  61% /boot
/dev/mapper/rl-home   50G  4.9G   43G  11% /home
tmpfs                363M     0  363M   0% /run/user/1000
/dev/mapper/rl-web    20G   24K   19G   1% /var/www
[cauchemar@node1 ~]$ sudo umount /var/www
[cauchemar@node1 ~]$ sudo mount /dev/rl/web /var/www -o noexec
[cauchemar@node1 ~]$ mount
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime,seclabel)
devtmpfs on /dev type devtmpfs (rw,nosuid,seclabel,size=4096k,nr_inodes=459000,mode=755,inode64)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev,seclabel,inode64)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,nodev,seclabel,size=742612k,nr_inodes=819200,mode=755,inode64)
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,seclabel,nsdelegate,memory_recursiveprot)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime,seclabel)
bpf on /sys/fs/bpf type bpf (rw,nosuid,nodev,noexec,relatime,mode=700)
/dev/mapper/rl-root on / type ext4 (rw,relatime,seclabel)
selinuxfs on /sys/fs/selinux type selinuxfs (rw,nosuid,noexec,relatime)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=29,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=17802)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,seclabel,pagesize=2M)
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime,seclabel)
debugfs on /sys/kernel/debug type debugfs (rw,nosuid,nodev,noexec,relatime,seclabel)
tracefs on /sys/kernel/tracing type tracefs (rw,nosuid,nodev,noexec,relatime,seclabel)
none on /run/credentials/systemd-sysctl.service type ramfs (ro,nosuid,nodev,noexec,relatime,seclabel,mode=700)
fusectl on /sys/fs/fuse/connections type fusectl (rw,nosuid,nodev,noexec,relatime)
configfs on /sys/kernel/config type configfs (rw,nosuid,nodev,noexec,relatime)
none on /run/credentials/systemd-tmpfiles-setup-dev.service type ramfs (ro,nosuid,nodev,noexec,relatime,seclabel,mode=700)
/dev/nvme0n1p2 on /var type ext4 (rw,relatime,seclabel)
/dev/nvme0n1p5 on /boot type ext4 (rw,relatime,seclabel)
/dev/mapper/rl-home on /home type ext4 (rw,relatime,seclabel)
none on /run/credentials/systemd-tmpfiles-setup.service type ramfs (ro,nosuid,nodev,noexec,relatime,seclabel,mode=700)
tmpfs on /run/user/1000 type tmpfs (rw,nosuid,nodev,relatime,seclabel,size=371304k,nr_inodes=92826,mode=700,uid=1000,gid=1000,inode64)
/dev/mapper/rl-web on /var/www type ext4 (rw,noexec,relatime,seclabel)
[cauchemar@node1 ~]$ cd /var/www/
[cauchemar@node1 www]$
[cauchemar@node1 www]$
[cauchemar@node1 www]$ echo "ls" > toto
-bash: toto: Permission denied
[cauchemar@node1 www]$ echo "ls" | sudo tee  toto
ls
[cauchemar@node1 www]$ cat toto
ls
[cauchemar@node1 www]$ sudo chmod 777 toto
[cauchemar@node1 www]$ ./toto
-bash: ./toto: Permission denied
[cauchemar@node1 www]$  81  sudo mount /dev/rl/web /var/www 81  sudo mount /dev/rl/web /var/wwws^C
[cauchemar@node1 www]$ ^C
[cauchemar@node1 www]$ sudo mount -o remount,ro /var/www
[cauchemar@node1 www]$ mount
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime,seclabel)
devtmpfs on /dev type devtmpfs (rw,nosuid,seclabel,size=4096k,nr_inodes=459000,mode=755,inode64)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev,seclabel,inode64)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,nodev,seclabel,size=742612k,nr_inodes=819200,mode=755,inode64)
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,seclabel,nsdelegate,memory_recursiveprot)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime,seclabel)
bpf on /sys/fs/bpf type bpf (rw,nosuid,nodev,noexec,relatime,mode=700)
/dev/mapper/rl-root on / type ext4 (rw,relatime,seclabel)
selinuxfs on /sys/fs/selinux type selinuxfs (rw,nosuid,noexec,relatime)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=29,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=17802)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,seclabel,pagesize=2M)
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime,seclabel)
debugfs on /sys/kernel/debug type debugfs (rw,nosuid,nodev,noexec,relatime,seclabel)
tracefs on /sys/kernel/tracing type tracefs (rw,nosuid,nodev,noexec,relatime,seclabel)
none on /run/credentials/systemd-sysctl.service type ramfs (ro,nosuid,nodev,noexec,relatime,seclabel,mode=700)
fusectl on /sys/fs/fuse/connections type fusectl (rw,nosuid,nodev,noexec,relatime)
configfs on /sys/kernel/config type configfs (rw,nosuid,nodev,noexec,relatime)
none on /run/credentials/systemd-tmpfiles-setup-dev.service type ramfs (ro,nosuid,nodev,noexec,relatime,seclabel,mode=700)
/dev/nvme0n1p2 on /var type ext4 (rw,relatime,seclabel)
/dev/nvme0n1p5 on /boot type ext4 (rw,relatime,seclabel)
/dev/mapper/rl-home on /home type ext4 (rw,relatime,seclabel)
none on /run/credentials/systemd-tmpfiles-setup.service type ramfs (ro,nosuid,nodev,noexec,relatime,seclabel,mode=700)
tmpfs on /run/user/1000 type tmpfs (rw,nosuid,nodev,relatime,seclabel,size=371304k,nr_inodes=92826,mode=700,uid=1000,gid=1000,inode64)
/dev/mapper/rl-web on /var/www type ext4 (ro,noexec,relatime,seclabel)
[cauchemar@node1 www]$

```
