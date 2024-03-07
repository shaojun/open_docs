## 查看某个时间段之后的`frps`的日志:
 ```
 journalctl -u frps.service --since=13:15  
 ```
## 某个`frps`进程所连接的TCP长连接数量:
 ```
 #每行仅匹配一次
 sudo netstat -tulp | grep frps -c
 #每行可匹配任意次
sudo netstat -tulp | grep frps | wc -l  
 ```
## 查看和临时(进程重启后作用将消失)更改某进程被设定的资源上限:
```
#以frp server的进程为例
#首先查看frp server的进程号:
(base) shao@ecs-01796520-002:~$ top | grep frp
3940392 root      20   0 1062492 101380   6372 S   6.2   0.6   1:03.00 frps
#可见为进程号, 即process id为`3940392`
#然后更改这个进程的 nofile(Max open files) 限制上限到 66666, 这也是经常导致所有边缘frp client连接到server 失败的原因:
prlimit -n66666 -p 3940392

## 用于查看某个进程的资源允许上限,应该可见这样的 Max open files            8096                 8096                 files
cat /proc/{pid_of_process}/limits
```

## 查看当前文件目录下的的最大size的文件夹: 
查看size最大的10个文件夹
```
du -aBM | sort -nr | head -n 10
```

查看size最大的10个文件夹, 并同时指定不搜索某个子文件夹 `./data`
```
du -aBM --exclude=./data 2>/dev/null | sort -nr | head -n 10
```

## 修改docker的默认存储路径:
```
sudo nano /lib/systemd/system/docker.service
#see the line of below, update that /data1/docker to your situation:
ExecStart=/usr/bin/dockerd --data-root /data1/docker -H fd:// --containerd=/run/containerd/containerd.sock
#then restart the docker service
sudo service docker restart
#after docker running a while, should see the disk usage under /data1/docker/ via
df -h
#root@ecs-01796520-002:~# df -h
#Filesystem      Size  Used Avail Use% Mounted on
#udev            7.8G     0  7.8G   0% /dev
#tmpfs           1.6G   58M  1.6G   4% /run
#/dev/sda1        40G   11G   28G  28% /
#tmpfs           7.9G  428K  7.9G   1% /dev/shm
#tmpfs           5.0M     0  5.0M   0% /run/lock
#tmpfs           7.9G     0  7.9G   0% /sys/fs/cgroup
#tmpfs           1.6G     0  1.6G   0% /run/user/1001
#/dev/sdb1       984G  2.3G  931G   1% /data1
#overlay         984G  2.3G  931G   1% /data1/docker/overlay2/bfe8b111430dae9507ace0d9953100be138e8a469b2fa3422e460fbb46b33873/merged
#overlay         984G  2.3G  931G   1% /data1/docker/overlay2/c4494c0c21ac17ddb0ab9152d0ce2a712bdd03435f11409b03618e0194bd7e8e/merged
#overlay         984G  2.3G  931G   1% /data1/docker/overlay2/6e073504fd36822f124c0f2ee5fd395d67147102e95bc97c51e62c3f8611d6d9/merged
#overlay         984G  2.3G  931G   1% /data1/docker/overlay2/eb5ca92e56022401a112e196f3628882925a760ec9eaddda9fdbd7573e8541e2/merged
#overlay         984G  2.3G  931G   1% /data1/docker/overlay2/a038a906664fab5be61355b777521b056f1d51b157ff9f8b3b9c2339849cdf01/merged
#shm              64M     0   64M   0% /data1/docker/containers/cdb8185f79b4a2c861da2c5821128a3a6029ece3f89da2d2a302dccb56890891/mounts/shm
#shm              64M     0   64M   0% /data1/docker/containers/e33d29ab60b89db283d066583b6871708c4fabb0ec79abeacd2a561437aeae46/mounts/shm
#shm              64M     0   64M   0% /data1/docker/containers/ddac674fdd4ccd103328387777ad0aa13d775e47a59d845e238cedd2b3691651/mounts/shm
#shm              64M     0   64M   0% /data1/docker/containers/76385eb23ab9854d3140117663cc0cc2b1ce472b6b023c01d5071af4a01eb7ed/mounts/shm
#shm              64M     0   64M   0% /data1/docker/containers/2d8fb198a307a627350be2e38d451eb3eb4778cd636a1ba2a99add23f4e90f8b/mounts/shm
#overlay         984G  2.3G  931G   1% /data1/docker/overlay2/3701b8e1eed4be282fe040be4d41b2db946c1fa6d1ad6fdbee145b9bca893a28/merged
#shm              64M     0   64M   0% /data1/docker/containers/1997b564ab1d106129dfcfc9ae4c7b89b6f1bccf9b07c4796c27ceb675eb4f46/mounts/shm

```

## 云主机系统盘扩容
这是未扩容前的系统盘磁盘`/dev/sda1`使用量, 可见其总容量只有40G, 而且已经使用了 **97%**:
```
(base) shao@ecs-01796520-002:~$ df -m
Filesystem     1M-blocks   Used Available Use% Mounted on
udev                7961      0      7961   0% /dev
tmpfs               1601    178      1424  12% /run
/dev/sda1          40253  37152      1345  97% /
tmpfs               8005      1      8005   1% /dev/shm
tmpfs                  5      0         5   0% /run/lock
tmpfs               8005      0      8005   0% /sys/fs/cgroup
/dev/sdb1        1006904 120130    835558  13% /data1
tmpfs               1601      0      1601   0% /run/user/1001
```
请在云主机管理后台对系统盘扩容付费和申请, 以下示例是在后台申请**新增加了110G**,即**总**的系统磁盘的容量是**150G**.

但因为新增加的空间还未更新至到`ubuntu 磁盘分区`信息中, 系统还无法直接使用此空间,但已经可以看到一个**更大的磁盘**:
```
(base) shao@ecs-01796520-002:~$ sudo fdisk -l
Disk /dev/sda: 150 GiB, 161061273600 bytes, 314572800 sectors
Disk model: QEMU HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xee566d79

Device     Boot Start      End  Sectors Size Id Type
/dev/sda1  *     2048 83886046 83883999  40G 83 Linux


....you may have other disks and will show below, omit in this tutorial....
```
可见 `Disk`: `/dev/sda` 的容量已经变成了  `150 GiB` (之前此磁盘是`40G`).

接下来对`磁盘`: `/dev/sda`进行分区信息查看和更新:

```
(base) shao@ecs-01796520-002:~$ sudo parted /dev/sda
GNU Parted 3.3
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.


```
`p free`用于查看当前目标磁盘上的分区信息:
```
(parted) p free
Model: QEMU QEMU HARDDISK (scsi)
Disk /dev/sda: 161GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
        1024B   1049kB  1048kB           Free Space
 1      1049kB  42.9GB  42.9GB  primary  ext4         boot
        42.9GB  161GB   118GB            Free Space
```
可见此磁盘仅有一个分区(仅`Number 1`), 也可看到有`118GB`是`Free Space`, 我们的工作就是让这块空闲空间`分配和并入`到`42.9GB`这个之前的`老分区`中去.

分区是由起始和结束扇区位置来定义的, 所以`分配和并入`空间就是更新之间`老分区`的起始和结束扇区即可, 先改变一下扇区位置的单位显示:
```
(parted) unit s
(parted) p free
Model: QEMU QEMU HARDDISK (scsi)
Disk /dev/sda: 314572800s
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start      End         Size        Type     File system  Flags
        2s         2047s       2046s                Free Space
 1      2048s      83886046s   83883999s   primary  ext4         boot
        83886047s  314572799s  230686753s           Free Space

```
可见单位变了(更精准).

然后开始更新扇区位置, 因此磁盘就1个分区,所以以下输入`1`:
```
(parted) resizepart 1
```
输入要并入的新容易的尾扇区号(可见默认的数值`83886046s`是之前40G容量的尾扇区位置):
```
Warning: Partition /dev/sda1 is being used. Are you sure you want to continue?
Yes/No? yes
End?  [83886046s]? 314572799s

```

再次查询分区信息,可见分区的尾扇区位置已经更新了:

```
(parted) p free
Model: QEMU QEMU HARDDISK (scsi)
Disk /dev/sda: 314572800s
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start  End         Size        Type     File system  Flags
        2s     2047s       2046s                Free Space
 1      2048s  314572799s  314570752s  primary  ext4         boot



```
按`q`退出:
```
(parted) q
```
此时用`df -m`仍然看不到最新的磁盘空间信息,还需要进行一次刷新:
```
(base) shao@ecs-01796520-002:~$ sudo resize2fs /dev/sda1
resize2fs 1.45.5 (07-Jan-2020)
Filesystem at /dev/sda1 is mounted on /; on-line resizing required
old_desc_blocks = 5, new_desc_blocks = 19
The filesystem on /dev/sda1 is now 39321344 (4k) blocks long.

```
最后确认:

```
(base) shao@ecs-01796520-002:~$ df -mh
Filesystem      Size  Used Avail Use% Mounted on
udev            7.8G     0  7.8G   0% /dev
tmpfs           1.6G  178M  1.4G  12% /run
/dev/sda1       148G   37G  106G  26% /
tmpfs           7.9G  544K  7.9G   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           7.9G     0  7.9G   0% /sys/fs/cgroup
/dev/sdb1       984G  118G  816G  13% /data1
tmpfs           1.6G     0  1.6G   0% /run/user/1001

```
可见 `/`已经扩容到`148G`.
## 挂载额外的数据盘到系统
1.查看所有硬盘信息
`fdisk -l` 查看所有硬盘
```
(base) shao@ecs-01796520-002:~$ sudo fdisk -l
[sudo] password for shao:
Disk /dev/sda: 150 GiB, 161061273600 bytes, 314572800 sectors
Disk model: QEMU HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xee566d79

Device     Boot Start       End   Sectors  Size Id Type
/dev/sda1  *     2048 314572799 314570752  150G 83 Linux


Disk /dev/sdb: 1000 GiB, 1073741824000 bytes, 2097152000 sectors
Disk model: QEMU HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x5bd527a5

Device     Boot Start        End    Sectors  Size Id Type
/dev/sdb1        2048 2097151999 2097149952 1000G 83 Linux

```
可见一个1000G容量的`Disk /dev/sdb` with 路径 `/dev/sdb1` 了.

2.创建挂载目录
按你自己的情况都行,比如:
```
mkdir /data
```

3. 直接挂载
```
mount /dev/sdb1 /data
```

4. 下次启动系统时自动挂载 
如果不加这步,下次重启系统后,挂载会消失(尽量新硬盘上的数据不会消失)
查看持载分区的UUID by `blkid /dev/sdb1`:
```
(base) shao@ecs-01796520-002:~$ blkid /dev/sdb1
/dev/sdb1: UUID="cbddb781-df6c-4c09-afb6-3730125be486" TYPE="ext4" PARTUUID="5bd527a5-01"
```
修改开机挂载文件:
```
nano /etc/fstab
#修改成类似内容

#shao pasted below for the 1000G disk of /dev/sdb1
#UUID="cbddb781-df6c-4c09-afb6-3730125be486" TYPE="ext4" PARTUUID="5bd527a5-01"
UUID=cbddb781-df6c-4c09-afb6-3730125be486 /data1 ext4 defautls 0 0
#shao done


```

