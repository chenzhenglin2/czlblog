## 磁盘扩展和缩减知识汇总



## 新增分区，挂靠到新的目录方法

- 1,首先通过命令lsblk  查看增加分区的情况；

  ```
  [root@apptrace0011 ~]# lsblk
  NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
  sda               8:0    0   501G  0 disk 
  ├─sda1            8:1    0     1G  0 part /boot
  ├─sda2            8:2    0   199G  0 part 
  │ ├─centos-root 253:0    0    50G  0 lvm  /
  │ ├─centos-swap 253:1    0   3.9G  0 lvm  [SWAP]
  │ └─centos-home 253:2    0 445.1G  0 lvm  /home
  └─sda3            8:3    0   301G  0 part 
    └─centos-home 253:2    0 445.1G  0 lvm  /home
  sdb               8:16   0    10G  0 disk 
  sr0              11:0    1  1024M  0 rom  
  ```

  关于sda/sdb说明，如果通过vmmare 虚拟机控制台等工具，直接在原有的1个硬盘扩充的存储空间；如原有硬盘是200G，
  扩充到500G扩充后，扩充的存储还是在sda分区下；如果新增一个硬盘，是在sdb分区，依次类推sdc……

  关于sda/sdb说明，如果通过vmmare 虚拟机控制台等工具，直接在原有的1个硬盘扩充的存储空间；如原有硬盘是200G，
  扩充到500G扩充后，扩充的存储还是在sda分区下；如果新增一个硬盘，是在sdb分区，依次类推sdc……

- 2，通过命令fdisk   -l  

  ```
   [root@apptrace0011 ~]# fdisk   -l  
  
  Disk /dev/sda: 537.9 GB, 537944653824 bytes, 1050673152 sectors
  Units = sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  Disk label type: dos
  Disk identifier: 0x000e999c
  
     Device Boot      Start         End      Blocks   Id  System
  /dev/sda1   *        2048     2099199     1048576    1  FAT12
  /dev/sda2         2099200   419430399   208665600   8e  Linux LVM
  /dev/sda3       419430400  1050673151   315621376   8e  Linux LVM
  
  Disk /dev/sdb: 10.7 GB, 10737418240 bytes, 20971520 sectors
  Units = sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  
  Disk /dev/mapper/centos-root: 53.7 GB, 53687091200 bytes, 104857600 sectors
  Units = sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  
  Disk /dev/mapper/centos-swap: 4160 MB, 4160749568 bytes, 8126464 sectors
  Units = sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  
  Disk /dev/mapper/centos-home: 477.9 GB, 477940940800 bytes, 933478400 sectors
  Units = sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  ```

  查看到sda 硬盘和sdb硬盘的情况，sda都已经做分区（但还有空间可以进行分区，下种类型讲）
  sdb还未有分区；如果新增了硬盘，没有看到可以执行命令：
  partprobe    /dev/sdb  ,没有这个命令，自行安装 yum  -y install   parted  

  查看到sda 硬盘和sdb硬盘的情况，sda都已经做分区（但还有空间可以进行分区，下种类型讲）
  sdb还未有分区；如果新增了硬盘，没有看到可以执行命令：
  partprobe    /dev/sdb  ,没有这个命令，自行安装 yum  -y install   parted  



- 3，如果新增硬盘在sdb下 可以按照如下方式直接挂载
  `fdisk /dev/sdb`
  输入m 查看用法 最常用几个用法 p 打印分区情况 n 新增分区； d删除分区；w保存 t改变格式
  输入p 打印分区情况
  输入 n新增分区
  Partition number (1-4, default 1): 正常情况默认选中1；，如果上步p打印时已经有sdb1，输入2
  然后输入t 改变分区格式

  ```
  Command (m for help): t
  Selected partition 1
  Partition type (type L to list all types): L
  0  Empty           24  NEC DOS         81  Minix / old Lin bf  Solaris        
   1  FAT12           27  Hidden NTFS Win 82  Linux swap / So c1  DRDOS/sec (FAT-
   2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
   3  XENIX usr       3c  PartitionMagic  84  OS/2 hidden or  c6  DRDOS/sec (FAT-
   4  FAT16 <32M      40  Venix 80286     85  Linux extended  c7  Syrinx         
   5  Extended        41  PPC PReP Boot   86  NTFS volume set da  Non-FS data    
   6  FAT16           42  SFS             87  NTFS volume set db  CP/M / CTOS / .
   7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux plaintext de  Dell Utility   
   8  AIX             4e  QNX4.x 2nd part 8e  Linux LVM       df  BootIt         
   9  AIX bootable    4f  QNX4.x 3rd part 93  Amoeba          e1  DOS access     
   a  OS/2 Boot Manag 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O        
   b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor      
   c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi ea  Rufus alignment
   e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         eb  BeOS fs        
   f  W95 Ext'd (LBA) 54  OnTrackDM6      a6  OpenBSD         ee  GPT            
  10  OPUS            55  EZ-Drive        a7  NeXTSTEP        ef  EFI (FAT-12/16/
  11  Hidden FAT12    56  Golden Bow      a8  Darwin UFS      f0  Linux/PA-RISC b
  12  Compaq diagnost 5c  Priam Edisk     a9  NetBSD          f1  SpeedStor      
  14  Hidden FAT16 <3 61  SpeedStor       ab  Darwin boot     f4  SpeedStor      
  16  Hidden FAT16    63  GNU HURD or Sys af  HFS / HFS+      f2  DOS secondary  
  17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fb  VMware VMFS    
  18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fc  VMware VMKCORE 
  1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fd  Linux raid auto
  1c  Hidden W95 FAT3 75  PC/IX           bc  Acronis FAT32 L fe  LANstep        
  1e  Hidden W95 FAT1 80  Old Minix       be  Solaris boot    ff  BBT     
  ```

     


选择8e  Linux LVM 这个格式，(有的是83、linux格式的)
最后输入w 保存退出（不能漏掉）

```
Partition type (type L to list all types): 8e      
Changed type of partition 'Linux' to 'Linux LVM'.
```

如果保存出现错误,可以 partprobe  /dev/sdb  (没有数字）
然后再进入 fdisk   /dev/sdb 继续上面的操作  甚至重启

如果保存出现错误,可以 partprobe  /dev/sdb  (没有数字）
然后再进入 fdisk   /dev/sdb 继续上面的操作  甚至重启



- 4，接着格式化：
  centos7 可以用`mkfs.xfs /dev/sdb1` ，Ubuntu或者centos6 用`mkfs.ext4 /dev/sdb1` 来格式
  输入mkfs.  按tab键，可以看出有哪些格式

  ```
  [root@apptrace0011 ~]# mkfs.
  mkfs.btrfs   mkfs.cramfs  mkfs.ext2    mkfs.ext3    mkfs.ext4    mkfs.minix   mkfs.xfs 
  ```

  


- 5，进行挂载：
  `mount /dev/sdb1  /data/（新目录或者老目录，如果没有需求提前创建）`

- 6，开机生效，编辑 /etc/fstab 

  ```
  /dev/mapper/centos-root /                       xfs     defaults        0 0
  UUID=3c94bedd-2b80-47d3-a3a4-05785847aa10 /boot                   xfs     defaults        0 0
  /dev/mapper/centos-home /home                   xfs     defaults        0 0
  /dev/mapper/centos-swap swap                    swap    defaults        0 0
  ```

  把刚才的/data 添加进去

  ```
  /dev/sdb1 /data                   xfs     defaults        0 0
  或者 UUID=3c94bedd-2b80-47d3-a3a4-05785847aa10 /data                xfs     defaults        0 0
  ```

  如果是在物理机上，增加硬盘后，最好填写uuid，分区是可以变化，uuid不会变；

  如果是在物理机上，增加硬盘后，最好填写uuid，分区是可以变化，uuid不会变；

  - 9.其他命令 blkid查看挂载硬盘的UUID，如blkid | grep  "sdb*" ，查看现有分区cat  /proc/partitions 

  把刚才的/data 添加进去

  ```
  /dev/sdb1 /data                   xfs     defaults        0 0
  或者 UUID=3c94bedd-2b80-47d3-a3a4-05785847aa10 /data                xfs     defaults        0 0
  ```

  如果是在物理机上，增加硬盘后，最好填写uuid，分区是可以变化，uuid不会变；

  - 8.其他命令 blkid查看挂载硬盘的UUID，如`blkid | grep  "sdb*" `，查看现有分区`cat  /proc/partitions `

  如果是在物理机上，增加硬盘后，最好填写uuid，分区是可以变化，uuid不会变；

  - 7.其他命令 blkid查看挂载硬盘的UUID，如`blkid | grep  "sdb*" `，查看现有分区`cat  /proc/partitions `

##  怎么把原有硬盘扩充的存储都挂靠到/home（或其他已有目录）

- 1，查看新增硬盘情况，如下，原有硬盘从200G增加到300G

  ```
  [root@part-add ~]# lsblk 
  NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
  sda               8:0    0   300G  0 disk 
  ├─sda1            8:1    0     1G  0 part /boot
  └─sda2            8:2    0   199G  0 part 
    ├─centos-root 253:0    0    50G  0 lvm  /
    ├─centos-swap 253:1    0   3.9G  0 lvm  [SWAP]
    └─centos-home 253:2    0 145.1G  0 lvm  /home
  sr0              11:0    1  1024M  0 rom  
  ```

  


再查看fdisk情况

```
[root@part-add ~]# fdisk -l

Disk /dev/sda: 322.1 GB, 322122547200 bytes, 629145600 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000e999c

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200   419430399   208665600   8e  Linux LVM

Disk /dev/mapper/centos-root: 53.7 GB, 53687091200 bytes, 104857600 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/centos-swap: 4160 MB, 4160749568 bytes, 8126464 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/centos-home: 155.8 GB, 155818393600 bytes, 304332800 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```



- 2，把增加的硬盘容量全部分到一个新分区sda3上

  ```
   fdisk  /dev/sda  
    Welcome to fdisk (util-linux 2.23.2).
  
  Changes will remain in memory only, until you decide to write them.
  Be careful before using the write command.
  
  Command (m for help): p
  
  Disk /dev/sda: 322.1 GB, 322122547200 bytes, 629145600 sectors
  Units = sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  Disk label type: dos
  Disk identifier: 0x000e999c
  
     Device Boot      Start         End      Blocks   Id  System
  /dev/sda1   *        2048     2099199     1048576   83  Linux
  /dev/sda2         2099200   419430399   208665600   8e  Linux LVM
  
  Command (m for help): n
  Partition type:
     p   primary (2 primary, 0 extended, 2 free)
     e   extended
  Select (default p): p
  Partition number (3,4, default 3): 
  First sector (419430400-629145599, default 419430400): 
  Using default value 419430400
  Last sector, +sectors or +size{K,M,G} (419430400-629145599, default 629145599): 
  Using default value 629145599
  Partition 3 of type Linux and of size 100 GiB is set
  
  Command (m for help): p
  
  Disk /dev/sda: 322.1 GB, 322122547200 bytes, 629145600 sectors
  Units = sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  Disk label type: dos
  Disk identifier: 0x000e999c
  
     Device Boot      Start         End      Blocks   Id  System
  /dev/sda1   *        2048     2099199     1048576   83  Linux
  /dev/sda2         2099200   419430399   208665600   8e  Linux LVM
  /dev/sda3       419430400   629145599   104857600   83  Linux
  
  Command (m for help): t
  Partition number (1-3, default 3): 
  Hex code (type L to list all codes): 8e  （注意lvm格式）
  Changed type of partition 'Linux' to 'Linux LVM'
  
  Command (m for help): p
  
  Disk /dev/sda: 322.1 GB, 322122547200 bytes, 629145600 sectors
  Units = sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  Disk label type: dos
  Disk identifier: 0x000e999c
  
     Device Boot      Start         End      Blocks   Id  System
  /dev/sda1   *        2048     2099199     1048576   83  Linux
  /dev/sda2         2099200   419430399   208665600   8e  Linux LVM
  /dev/sda3       419430400   629145599   104857600   8e  Linux LVM
  
  Command (m for help): w   
  The partition table has been altered!
  
  Calling ioctl() to re-read partition table.
  
  WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
  The kernel still uses the old table. The new table will be used at
  the next reboot or after you run partprobe(8) or kpartx(8)
  Syncing disks.
  [root@part-add ~]# partprobe  /dev/sda
  [root@part-add ~]# partprobe  /dev/sda3
  [root@part-add ~]# fdisk -l
  
  Disk /dev/sda: 322.1 GB, 322122547200 bytes, 629145600 sectors
  Units = sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  Disk label type: dos
  Disk identifier: 0x000e999c
  
     Device Boot      Start         End      Blocks   Id  System
  /dev/sda1   *        2048     2099199     1048576   83  Linux
  /dev/sda2         2099200   419430399   208665600   8e  Linux LVM
  /dev/sda3       419430400   629145599   104857600   8e  Linux LVM
  
  Disk /dev/mapper/centos-root: 53.7 GB, 53687091200 bytes, 104857600 sectors
  Units = sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  
  Disk /dev/mapper/centos-swap: 4160 MB, 4160749568 bytes, 8126464 sectors
  Units = sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  
  Disk /dev/mapper/centos-home: 155.8 GB, 155818393600 bytes, 304332800 sectors
  Units = sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  ```

  




- 3，新增分区格式化（执行顺序可以和下面第4部互换）

  ```
  mkfs.xfs   /dev/sda3 
  meta-data=/dev/sda3              isize=512    agcount=4, agsize=6553600 blks
           =                       sectsz=512   attr=2, projid32bit=1
           =                       crc=1        finobt=0, sparse=0
  data     =                       bsize=4096   blocks=26214400, imaxpct=25
           =                       sunit=0      swidth=0 blks
  naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
  log      =internal log           bsize=4096   blocks=12800, version=2
           =                       sectsz=512   sunit=0 blks, lazy-count=1
  realtime =none                   extsz=4096   blocks=0, rtextents=0
  ```

  




- 4，增加物理卷：  pvcreate 刚才创建的分区

  ```
  [root@part-add ~]# pvcreate  /dev/sda3  
  WARNING: xfs signature detected on /dev/sda3 at offset 0. Wipe it? [y/n]: y
    Wiping xfs signature on /dev/sda3.
    Physical volume "/dev/sda3" successfully created.
  ```

  查看物理卷增加后的情况

  ```
   [root@part-add ~]# pvdisplay 或者pvs
      --- Physical volume ---
      PV Name               /dev/sda2
      VG Name               centos
      PV Size               <199.00 GiB / not usable 3.00 MiB
      Allocatable           yes 
      PE Size               4.00 MiB
      Total PE              50943
      Free PE               1
      Allocated PE          50942
      PV UUID               sa1cah-eS6t-c5sr-RnYs-PMhE-4CCx-Of0Fnl
  
    "/dev/sda3" is a new physical volume of "100.00 GiB"
    --- NEW Physical volume ---
    PV Name               /dev/sda3
    VG Name               
    PV Size               100.00 GiB
    Allocatable           NO
    PE Size               0   
    Total PE              0
    Free PE               0
    Allocated PE          0
    PV UUID               sYiCZM-0zBg-6yF0-tiN3-k8PL-2y9w-rTODx0
  ```

  

  

- 5，将物理卷加入到卷组

  * 1）先看卷组信息

    ```
    [root@part-add ~]# vgdisplay 或者vgs
      --- Volume group ---
      VG Name               centos
      System ID             
      Format                lvm2
      Metadata Areas        1
      Metadata Sequence No  4
      VG Access             read/write
      VG Status             resizable
      MAX LV                0
      Cur LV                3
      Open LV               2
      Max PV                0
      Cur PV                1
      Act PV                1
      VG Size               <199.00 GiB
      PE Size               4.00 MiB
      Total PE              50943
      Alloc PE / Size       50942 / 198.99 GiB
      Free  PE / Size       1 / 4.00 MiB
      VG UUID               iza5sY-YoTh-ihTR-dLzZ-2CBx-M0dR-33bzW5
    ```

    


  * 2）把新的分区加入到卷组 vgextend centos（VG Name） /dev/sda3  （新分区）

    * ```
      [root@part-add ~]# vgextend centos /dev/sda3
        Volume group "centos" successfully extended
      
      - 3）再次查看 验证
        [root@part-add ~]# vgdisplay
          --- Volume group ---
          VG Name               centos
          System ID             
          Format                lvm2
          Metadata Areas        2
          Metadata Sequence No  5
          VG Access             read/write
          VG Status             resizable
          MAX LV                0
          Cur LV                3
          Open LV               2
          Max PV                0
          Cur PV                2
          Act PV                2
          VG Size               298.99 GiB
          PE Size               4.00 MiB
          Total PE              76542
          Alloc PE / Size       50942 / 198.99 GiB
          Free  PE / Size       25600 / 100.00 GiB
          VG UUID               iza5sY-YoTh-ihTR-dLzZ-2CBx-M0dR-33bzW5
      ```


        此时VG Size 大小已有 298.99 GiB  

- 6，扩充逻辑卷  

  - 1）先通过下面命令查看系统里有哪些逻辑卷。

    ```
     [root@part-add ~]# lvdisplay  或者lvs
          --- Logical volume ---
          LV Path                /dev/centos/swap
          LV Name                swap
          VG Name                centos
          LV UUID                m8Dc95-LYbI-cxuG-hNc7-COoH-4Axh-7mamwX
          LV Write Access        read/write
          LV Creation host, time localhost, 2018-08-31 18:38:32 +0800
          LV Status              available
    
      LV Size                <3.88 GiB
      Current LE             992
      Segments               1
      Allocation             inherit
      Read ahead sectors     auto
    
    - currently set to     8192
      Block device           253:1
    
      --- Logical volume ---
      LV Path                /dev/centos/home
      LV Name                home
      VG Name                centos
      LV UUID                1HItab-O0cU-CEkB-cplf-6axH-utqI-Fgxi5n
      LV Write Access        read/write
      LV Creation host, time localhost, 2018-08-31 18:38:33 +0800
      LV Status              available
    
    
    
      LV Size                <145.12 GiB
      Current LE             37150
      Segments               1
      Allocation             inherit
      Read ahead sectors     auto
    
    - currently set to     8192
      Block device           253:2
    
      --- Logical volume ---
      LV Path                /dev/centos/root
      LV Name                root
      VG Name                centos
      LV UUID                2AdRQf-qOJb-h32O-SbVq-74yx-yic2-b8TyAR
      LV Write Access        read/write
      LV Creation host, time localhost, 2018-08-31 18:38:34 +0800
      LV Status              available
    
      LV Size                50.00 GiB
      Current LE             12800
      Segments               1
      Allocation             inherit
      Read ahead sectors     auto
    
    - currently set to     8192
      Block device           253:0  
    ```

    


有/dev/centos/swap /dev/centos/home /dev/centos/root这三个逻辑卷,  其中逻辑卷/dev/centos/home （挂载点是home目录下）就是本次要扩充的对象（同理根目录/ 对应的  /dev/centos/root也可以安装此方法)  

- 2)扩充逻辑卷/dev/centos/home
  lvextend -L  +100G  /dev/mapper/centos-home
  或者：lvextend -l  提示数量（可以查看Current LE，如果提示太多，减少到提示的最多数量）   /dev/mapper/centos-home
  lvextend -l  +100%FREE  /dev/mapper/centos-home

  ```
  [root@part-add ~]# lvextend -L  +100G  /dev/mapper/centos-home
    Size of logical volume centos/home changed from <145.12 GiB (37150 extents) to <245.12 GiB (62750 extents).
    Logical volume centos/home successfully resized.
  ```

  

-  3）扩充到文件系统（目录）中，xfs_growfs   /dev/centos/home    
    如果是ext格式  则用resize2fs /dev/centos/home

  ```
  [root@part-add ~]# xfs_growfs   /dev/centos/home 
  meta-data=/dev/mapper/centos-home isize=512    agcount=4, agsize=9510400 blks
           =                       sectsz=512   attr=2, projid32bit=1
           =                       crc=1        finobt=0 spinodes=0
  data     =                       bsize=4096   blocks=38041600, imaxpct=25
           =                       sunit=0      swidth=0 blks
  naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
  log      =internal               bsize=4096   blocks=18575, version=2
           =                       sectsz=512   sunit=0 blks, lazy-count=1
  realtime =none                   extsz=4096   blocks=0, rtextents=0
  data blocks changed from 38041600 to 64256000
  [root@part-add ~]# df -h
  Filesystem               Size  Used Avail Use% Mounted on
  /dev/mapper/centos-root   50G  1.5G   49G   3% /
  devtmpfs                 1.9G     0  1.9G   0% /dev
  tmpfs                    1.9G     0  1.9G   0% /dev/shm
  tmpfs                    1.9G  8.9M  1.9G   1% /run
  tmpfs                    1.9G     0  1.9G   0% /sys/fs/cgroup
  /dev/sda1               1014M  184M  831M  19% /boot
  tmpfs                    380M     0  380M   0% /run/user/0
  /dev/mapper/centos-home  246G   33M  246G   1% /home  
  ```

  


- 重启后再查看 df -h ，是扩充成功后的；一般不需重启。但最好mount -a 生效一下

## 新增磁盘挂载步骤概述

如何给服务器增加三块硬盘 ：

###    1，将三块硬盘增加到pv（物理卷）

   `pvcreate /dev/sdb /dev/sdc /dev/sdd`

###    2,将pv加入到vg（卷）组

  ` vgcreate  datavg  /dev/sdb /dev/sdc /dev/sdd`

###    3,分配逻辑卷

```
   lvcreate  -l 50%FREE -n lv1 datavg
   lvcreate  -L +200M -n lv2 datavg
```



###    4,格式化逻辑卷

 

```
  mkfs.xfs /dev/datavg/lv1
  mkfs.ext4  /dev/datavg/lv2
```



###    5,挂载

```
   mkdir  /lv1  /lv2
   mount /dev/datavg/lv1   /lv1
   mount /dev/datavg/lv2  /lv2
```



## 扩展磁盘步骤概述

###      1，添加物理磁盘

​	` pvcreate /dev/sdd`

### 	 2，扩展到现有vg组

​	` vgextend 现有卷组名称 /dev/sdd`

### 	 3，扩充到现有逻辑卷中

​	 

```
lvextend -L  +100G  /dev/mapper/centos-home
lvextend -l  +100%FREE(或者扩展数量)  /dev/mapper/centos-home
```



### 	 4，扩充到文件系统中

​	 

```
xfs_growfs   /dev/centos/home
resize2fs /dev/centos/home
```

​	 

## vg组减小和迁移等步骤概述

### 	1，减小vg

​	`vgremove  vg组名  /dev/sdd (或者其他要移动物理卷)`

### 	2，迁移vg

​	迁移vg里面的物理卷，必须是在同一个vg组中
​	`vgmove /dev/sdb   /dev/sdc (在线迁移)   `