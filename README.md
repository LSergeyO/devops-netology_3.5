# devops-netology_3.5  
1. lsergeyo@LSergeyO:~$ truncate -s1G file-sparse  
lsergeyo@LSergeyO:~$ stat file-sparse   
  - Файл: file-sparse  
  - Размер: 1073741824	Блоков: 0          Блок В/В: 4096   обычный файл
  - Устройство: 812h/2066d	Инода: 917751      Ссылки: 1
  - Доступ: (0664/-rw-rw-r--)  Uid: ( 1000/lsergeyo)   Gid: ( 1000/lsergeyo)
  - Доступ:        2022-01-31 21:41:57.184218624 +0800
  - Модифицирован: 2022-01-31 21:41:57.184218624 +0800
  - Изменён:       2022-01-31 21:41:57.184218624 +0800
  - Создан:        -  
2. Нет, не могут, так имеют один и тот же дескриптор  
3. root@vagrant:~# fdisk -l  
- Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
- Disk model: VBOX HARDDISK   
- Units: sectors of 1 * 512 = 512 bytes
- Sector size (logical/physical): 512 bytes / 512 bytes
- I/O size (minimum/optimal): 512 bytes / 512 bytes  

- Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
- Disk model: VBOX HARDDISK   
- Units: sectors of 1 * 512 = 512 bytes
- Sector size (logical/physical): 512 bytes / 512 bytes
- I/O size (minimum/optimal): 512 bytes / 512 bytes  
4. fdisk /dev/sdb  
- Device     Boot   Start     End Sectors  Size Id Type  
- /dev/sdb1          2048 4196351 4194304    2G 83 Linux  
- /dev/sdb2       4196352 5242879 1046528  511M 83 Linux  
5. root@vagrant:~# sfdisk -d /dev/sdb > part_table  
root@vagrant:~# sfdisk /dev/sdc < part_table  
- Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
- Disk model: VBOX HARDDISK   
- Units: sectors of 1 * 512 = 512 bytes
- Sector size (logical/physical): 512 bytes / 512 bytes
- I/O size (minimum/optimal): 512 bytes / 512 bytes
- Disklabel type: dos
- Disk identifier: 0x10f82459

- Device     Boot   Start     End Sectors  Size Id Type
- /dev/sdc1          2048 4196351 4194304    2G 83 Linux
- /dev/sdc2       4196352 5242879 1046528  511M 83 Linux
6. mdadm --create --verbose /dev/md0 -l 1 -n 2 /dev/sd{b,c}1  
root@vagrant:~# lsblk  
- NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
- sdb                         8:16   0  2.5G  0 disk  
- ├─sdb1                      8:17   0    2G  0 part  
- │ └─md0                     9:0    0    2G  0 raid1 
- └─sdb2                      8:18   0  511M  0 part  
- sdc                         8:32   0  2.5G  0 disk  
- ├─sdc1                      8:33   0    2G  0 part  
- │ └─md0                     9:0    0    2G  0 raid1 
- └─sdc2                      8:34   0  511M  0 part  
7. mdadm --create --verbose /dev/md1 -l 0 -n 2 /dev/sd{b,c}2  
root@vagrant:~# lsblk
- NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
- sdb                         8:16   0  2.5G  0 disk  
- ├─sdb1                      8:17   0    2G  0 part  
- │ └─md0                     9:0    0    2G  0 raid1 
- └─sdb2                      8:18   0  511M  0 part  
- └─md1                     9:1    0 1018M  0 raid0 
- sdc                         8:32   0  2.5G  0 disk  
- ├─sdc1                      8:33   0    2G  0 part  
- │ └─md0                     9:0    0    2G  0 raid1 
- └─sdc2                      8:34   0  511M  0 part  
- └─md1                     9:1    0 1018M  0 raid0  
8. root@vagrant:~# pvcreate /dev/md0  
  - Physical volume "/dev/md0" successfully created.  
root@vagrant:~# pvcreate /dev/md1  
  - Physical volume "/dev/md1" successfully created.  
9. root@vagrant:~# vgcreate vg01 /dev/md0 /dev/md1  
  - Volume group "vg01" successfully created  
root@vagrant:~# vgdisplay  

  - --- Volume group ---
  - VG Name               vg01
  - System ID             
  - Format                lvm2
  - Metadata Areas        2
  - Metadata Sequence No  1
  - VG Access             read/write
  - VG Status             resizable
  - MAX LV                0
  - Cur LV                0
  - Open LV               0
  - Max PV                0
  - Cur PV                2
  - Act PV                2
  - VG Size               <2.99 GiB
  - PE Size               4.00 MiB
  - Total PE              765
  - Alloc PE / Size       0 / 0   
  - Free  PE / Size       765 / <2.99 GiB
  - VG UUID               tc76lk-wsH4-F5jS-jqtM-SSkN-OJri-mK7bDc  
10. root@vagrant:~# lvcreate -L 100m vg01 /dev/md1  
 - Logical volume "lvol0" created.  
root@vagrant:~# lvs -o +devices  
 - LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert Devices     
 - ubuntu-lv ubuntu-vg -wi-ao----  31.50g                                                     /dev/sda3(0)
 - lvol0     vg01      -wi-a----- 100.00m                                                     /dev/md1(0)   
11. root@vagrant:~# mkfs.ext4 /dev/vg01/lvol0  
- mke2fs 1.45.5 (07-Jan-2020)
- Creating filesystem with 25600 4k blocks and 25600 inodes
- Allocating group tables: done                            
- Writing inode tables: done                            
- Creating journal (1024 blocks): done
- Writing superblocks and filesystem accounting information: done  
12. root@vagrant:~# mount /dev/vg01/lvol0 /tmp/new/  
root@vagrant:~# df -hT  
- Filesystem                        Type      Size  Used Avail Use% Mounted on
- /dev/mapper/vg01-lvol0            ext4       93M   72K   86M   1% /tmp/new  
13. root@vagrant:/tmp/new# ls  
- lost+found  test.gz  
14. root@vagrant:/tmp/new# lsblk  
- NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
- loop0                       7:0    0 55.4M  1 loop  /snap/core18/2128
- loop1                       7:1    0 70.3M  1 loop  /snap/lxd/21029
- loop2                       7:2    0 32.3M  1 loop  /snap/snapd/12704
- loop3                       7:3    0 55.5M  1 loop  /snap/core18/2284
- loop4                       7:4    0 43.4M  1 loop  /snap/snapd/14549
- loop5                       7:5    0 61.9M  1 loop  /snap/core20/1328
- loop6                       7:6    0 67.2M  1 loop  /snap/lxd/21835
- sda                         8:0    0   64G  0 disk  
- ├─sda1                      8:1    0    1M  0 part  
- ├─sda2                      8:2    0    1G  0 part  /boot
- └─sda3                      8:3    0   63G  0 part  
- └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm   /
- sdb                         8:16   0  2.5G  0 disk  
- ├─sdb1                      8:17   0    2G  0 part  
- │ └─md0                     9:0    0    2G  0 raid1 
- └─sdb2                      8:18   0  511M  0 part  
- └─md1                     9:1    0 1018M  0 raid0 
- └─vg01-lvol0          253:1    0  100M  0 lvm   /tmp/new
- sdc                         8:32   0  2.5G  0 disk  
- ├─sdc1                      8:33   0    2G  0 part  
- │ └─md0                     9:0    0    2G  0 raid1 
- └─sdc2                      8:34   0  511M  0 part  
- └─md1                     9:1    0 1018M  0 raid0 
- └─vg01-lvol0          253:1    0  100M  0 lvm   /tmp/new  
15.  root@vagrant:/tmp/new# gzip -t /tmp/new/test.gz  
root@vagrant:/tmp/new# echo $?  
- 0  
16. root@vagrant:/tmp/new# pvmove /dev/md1 /dev/md0  
- /dev/md1: Moved: 16.00%
- /dev/md1: Moved: 100.00%  
root@vagrant:/tmp/new# lsblk  
- NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
- sdc                         8:32   0  2.5G  0 disk  
- ├─sdc1                      8:33   0    2G  0 part  
- │ └─md0                     9:0    0    2G  0 raid1 
- │   └─vg01-lvol0          253:1    0  100M  0 lvm   /tmp/new
- └─sdc2                      8:34   0  511M  0 part  
- └─md1                     9:1    0 1018M  0 raid0  
17. root@vagrant:/tmp/new# mdadm --fail /dev/md0 /dev/sdb1  
- mdadm: set /dev/sdb1 faulty in /dev/md0  
18. root@vagrant:/tmp/new# dmesg | grep md0  
- [  421.709330] md/raid1:md0: not clean -- starting background reconstruction
- [  421.709333] md/raid1:md0: active with 2 out of 2 mirrors
- [  421.709356] md0: detected capacity change from 0 to 2144337920
- [  421.715297] md: resync of RAID array md0
- [  432.463152] md: md0: resync done.
- [ 4007.759380] md/raid1:md0: Disk failure on sdb1, disabling device. 
- md/raid1:md0: Operation continuing on 1 devices.  
19. root@vagrant:/tmp/new# gzip -t /tmp/new/test.gz  
root@vagrant:/tmp/new# echo $?  
- 0
20. lsergeyo@LSergeyO:~/vagrant$ sudo vagrant destroy  
- default: Are you sure you want to destroy the 'default' VM? [y/N] y
- ==> default: Destroying VM and associated drives...






















