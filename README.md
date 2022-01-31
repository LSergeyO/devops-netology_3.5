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
7. 






