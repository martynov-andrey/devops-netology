# devops-netology

---

### Домашнее задание к занятию "3.5. Файловые системы"

1. #### Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.

```bash
    $ truncate -s 512M file.img
    $ du -h --apparent-size file.img
512M    file.img
    $ du -h file.img
0       file.img
    $ cp file.img copy.img --sparse=never
    $ du -h copy.img
512M    copy.img
    $ fallocate -d copy.img && du -h copy.img
0       copy.img
    $ cp file.img sparse_copy.img --sparse=always
    $ du -h sparse_copy.img
0       sparse_copy.img
````

2. #### Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?

```bash
   $ touch test_file
   $ ln test_file test_file_hl1
   $ chmod +x test_file_hl1
   vagrant@vagrant:~$ ls -l test_file*
-rwxrwxr-x 2 vagrant vagrant 0 Dec  7 15:08 test_file
-rwxrwxr-x 2 vagrant vagrant 0 Dec  7 15:08 test_file_hl1
   $ sudo chown root. test_file_hl1
   $ ls -l test_file*
-rwxrwxr-x 2 root root 0 Dec  7 15:08 test_file
-rwxrwxr-x 2 root root 0 Dec  7 15:08 test_file_hl1
```

Созданный файл `test_file` и hardlink на него `test_file_hl1`, являются идентичными объектами файловой системы. Для них нельзя выставить отличные от файла права или владельца. 

3. #### Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```bash
    Vagrant.configure("2") do |config|
      config.vm.box = "bento/ubuntu-20.04"
      config.vm.provider :virtualbox do |vb|
        lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
        lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
        vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
        vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
      end
    end
    ```
    
Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.

```bash
vagrant@vagrant:~$ lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
sdb                    8:16   0  2.5G  0 disk
sdc                    8:32   0  2.5G  0 disk
```

4. #### Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.

```bash
   $ sudo fdisk -l /dev/sdb
Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xe44b54ab

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdb1          2048 4196351 4194304    2G 83 Linux
/dev/sdb2       4196352 5242879 1046528  511M 83 Linux
```

5. #### Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.

```bash
   $ sudo sfdisk -d /dev/sdb > sdb.dump
   $ sudo sfdisk /dev/sdc < sdb.dump
Checking that no-one is using this disk right now ... OK

Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xe44b54ab

Old situation:

Device     Boot Start     End Sectors Size Id Type
/dev/sdc1        2048 4196351 4194304   2G 83 Linux

>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Created a new DOS disklabel with disk identifier 0xe44b54ab.
/dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.
/dev/sdc2: Created a new partition 2 of type 'Linux' and of size 511 MiB.
/dev/sdc3: Done.

New situation:
Disklabel type: dos
Disk identifier: 0xe44b54ab

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1          2048 4196351 4194304    2G 83 Linux
/dev/sdc2       4196352 5242879 1046528  511M 83 Linux

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

6. Соберите `mdadm` RAID1 на паре разделов 2 Гб.

```bash
   $ sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sd[bc]1
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array? Y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
  $ cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid1 sdc1[1] sdb1[0]
      2094080 blocks super 1.2 [2/2] [UU]
unused devices: <none>
```

7. #### Соберите `mdadm` RAID0 на второй паре маленьких разделов.

```bash
   $ sudo mdadm --create /dev/md1 --level=0 --raid-devices=2 /dev/sd[bc]2
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
   $ cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md1 : active raid0 sdc2[1] sdb2[0]
      1042432 blocks super 1.2 512k chunks

md0 : active raid1 sdc1[1] sdb1[0]
      2094080 blocks super 1.2 [2/2] [UU]
unused devices: <none>
```

8. #### Создайте 2 независимых PV на получившихся md-устройствах.

```bash
   $ sudo pvcreate /dev/md0 /dev/md1
  Physical volume "/dev/md0" successfully created.
  Physical volume "/dev/md1" successfully created.
   $ sudo pvdisplay /dev/md[0,1]
  --- Physical volume ---
  PV Name               /dev/md0
  VG Name               VG0
  PV Size               <2.00 GiB / not usable 0
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              511
  Free PE               511
  Allocated PE          0
  PV UUID               Rjx6YB-vmqQ-gXor-DYr5-RDhr-2rlY-7y36Xe

  --- Physical volume ---
  PV Name               /dev/md1
  VG Name               VG0
  PV Size               1018.00 MiB / not usable 2.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              254
  Free PE               254
  Allocated PE          0
  PV UUID               z8HqE6-Vq8d-JOcv-a7Qc-89wy-uMJ2-bK3pab
```

9. #### Создайте общую volume-group на этих двух PV.

```bash
   $ sudo vgcreate -v VG0 /dev/md[0,1]
  Wiping signatures on new PV /dev/md0.
  Wiping signatures on new PV /dev/md1.
  Adding physical volume '/dev/md0' to volume group 'VG0'
  Adding physical volume '/dev/md1' to volume group 'VG0'
  Archiving volume group "VG0" metadata (seqno 0).
  Creating volume group backup "/etc/lvm/backup/VG0" (seqno 1).
  Volume group "VG0" successfully created
  $ sudo vgdisplay VG0
  --- Volume group ---
  VG Name               VG0
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               <2.99 GiB
  PE Size               4.00 MiB
  Total PE              765
  Alloc PE / Size       0 / 0
  Free  PE / Size       765 / <2.99 GiB
  VG UUID               Js65eh-9Bgl-JntR-bUPt-jawv-KEpm-4gJOWO
```

10. #### Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.

```bash
   $ sudo lvcreate -L 100m -n LV0 VG0 /dev/md1
  Logical volume "LV0" created.
   $ sudo lvdisplay VG0
  --- Logical volume ---
  LV Path                /dev/VG0/LV0
  LV Name                LV0
  VG Name                VG0
  LV UUID                X1Y1Iy-YIVu-wXPO-fzfc-152K-a3Qb-vGzmQq
  LV Write Access        read/write
  LV Creation host, time vagrant, 2021-12-07 17:47:57 +0000
  LV Status              available
  # open                 0
  LV Size                100.00 MiB
  Current LE             25
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     4096
  Block device           253:2
```

11. #### Создайте `mkfs.ext4` ФС на получившемся LV.

```bash
   $ sudo mkfs.ext4 /dev/VG0/LV0
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
```

12. #### Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.

```bash
   $ mkdir /tmp/new
   $ sudo mount /dev/VG0/LV0 /tmp/new
   $ mount | grep new
/dev/mapper/VG0-LV0 on /tmp/new type ext4 (rw,relatime,stripe=256)
```

13. #### Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.

```bash
   $ sudo wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
--2021-12-07 20:01:04--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 22739144 (22M) [application/octet-stream]
Saving to: ‘/tmp/new/test.gz’

/tmp/new/test.gz                                        100%[==============================================================================================================================>]  21.69M  6.28MB/s    in 3.6s

2021-12-07 20:01:08 (6.06 MB/s) - ‘/tmp/new/test.gz’ saved [22739144/22739144]
```

14. #### Прикрепите вывод `lsblk`.

```bash
  $ lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part  /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
sdb                    8:16   0  2.5G  0 disk
├─sdb1                 8:17   0    2G  0 part
│ └─md0                9:0    0    2G  0 raid1
└─sdb2                 8:18   0  511M  0 part
  └─md1                9:1    0 1018M  0 raid0
    └─VG0-LV0        253:2    0  100M  0 lvm   /tmp/new
sdc                    8:32   0  2.5G  0 disk
├─sdc1                 8:33   0    2G  0 part
│ └─md0                9:0    0    2G  0 raid1
└─sdc2                 8:34   0  511M  0 part
  └─md1                9:1    0 1018M  0 raid0
    └─VG0-LV0        253:2    0  100M  0 lvm   /tmp/new
```

15. #### Протестируйте целостность файла:

     ```bash
     root@vagrant:~# gzip -t /tmp/new/test.gz
     root@vagrant:~# echo $?
     0
     ```
```bash
    $ sudo gzip -t /tmp/new/test.gz
    $ sudo echo $?
0
```

16. #### Используя pvmove, переместите содержимое PV с RAID0 на RAID1.

```bash
  $ sudo pvmove -v /dev/md1 /dev/md0
  Executing: /sbin/modprobe dm-mirror
  Archiving volume group "VG0" metadata (seqno 2).
  Creating logical volume pvmove0
  activation/volume_list configuration setting not defined: Checking only host tags for VG0/LV0.
  Moving 25 extents of logical volume VG0/LV0.
  activation/volume_list configuration setting not defined: Checking only host tags for VG0/LV0.
  Creating VG0-pvmove0
  Loading table for VG0-pvmove0 (253:3).
  Loading table for VG0-LV0 (253:2).
  Suspending VG0-LV0 (253:2) with device flush
  Resuming VG0-pvmove0 (253:3).
  Resuming VG0-LV0 (253:2).
  Creating volume group backup "/etc/lvm/backup/VG0" (seqno 3).
  activation/volume_list configuration setting not defined: Checking only host tags for VG0/pvmove0.
  Checking progress before waiting every 15 seconds.
  /dev/md1: Moved: 24.00%
  /dev/md1: Moved: 100.00%
  $ lsblk /dev/sd[b,c]
NAME          MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sdb             8:16   0  2.5G  0 disk  
├─sdb1          8:17   0    2G  0 part  
│ └─md0       9:126  0    2G  0 raid1 
│   └─VG0-LV0 253:2    0  100M  0 lvm   
└─sdb2          8:18   0  511M  0 part  
  └─md1       9:127  0 1018M  0 raid0 
sdc             8:32   0  2.5G  0 disk  
├─sdc1          8:33   0    2G  0 part  
│ └─md0       9:126  0    2G  0 raid1 
│   └─VG0-LV0 253:2    0  100M  0 lvm   
└─sdc2          8:34   0  511M  0 part  
  └─md1       9:127  0 1018M  0 raid0
```

17. #### Сделайте `--fail` на устройство в вашем RAID1 md.

```bash
    $ sudo mdadm --fail /dev/md0 /dev/sdb1
mdadm: set /dev/sdb1 faulty in /dev/md0
    $ sudo mdadm -D /dev/md0
/dev/md126:
           Version : 1.2
     Creation Time : Tue Dec  7 16:59:25 2021
        Raid Level : raid1
        Array Size : 2094080 (2045.00 MiB 2144.34 MB)
     Used Dev Size : 2094080 (2045.00 MiB 2144.34 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Wed Dec  8 11:07:26 2021
             State : clean, degraded 
    Active Devices : 1
   Working Devices : 1
    Failed Devices : 1
     Spare Devices : 0

Consistency Policy : resync

              Name : vagrant:0  (local to host vagrant)
              UUID : 856dac6a:32e1ee04:866e48a2:4efb694f
            Events : 24

    Number   Major   Minor   RaidDevice State
       -       0        0        0      removed
       1       8       33        1      active sync   /dev/sdc1

       0       8       17        -      faulty   /dev/sdb1
```

18. #### Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.

```bash
    $ dmesg | grep md0
[   11.753298] md/raid1:md0: active with 2 out of 2 mirrors
[   11.753345] md0: detected capacity change from 0 to 2144337920
[  830.046920] md/raid1:md0: Disk failure on sdb1, disabling device.
               md/raid1:md0: Operation continuing on 1 devices.
```    

19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

     ```bash
     root@vagrant:~# gzip -t /tmp/new/test.gz
     root@vagrant:~# echo $?
     0
     ```
```bash
    $ sudo gzip -t /tmp/new/test.gz
    $ sudo echo $?
0
```

20. #### Погасите тестовый хост, `vagrant destroy`.

```bash
    $ vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
```

---
