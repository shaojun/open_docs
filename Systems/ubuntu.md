查看某个时间段之后的`frps`的日志:
 ```
 journalctl -u frps.service --since=13:15  
 ```
某个`frps`进程所连接的TCP长连接数量:
 ```
 #每行仅匹配一次
 sudo netstat -tulp | grep frps -c
 #每行可匹配任意次
sudo netstat -tulp | grep frps | wc -l  
 ```
查看和临时(进程重启后作用将消失)更改某进程被设定的资源上限:
```
#以frp server的进程为例
#首先查看frp server的进程号:
(base) shao@ecs-01796520-002:~$ top | grep frp
3940392 root      20   0 1062492 101380   6372 S   6.2   0.6   1:03.00 frps
#可见为进程号, 即process id为`3940392`
#然后更改这个进程的 nofile(Max open files) 限制上限到 66666, 这也是经常导致所有边缘frp client连接到server 失败的原因:
prlimit -n66666 -p 3940392

#用于查看某个进程的资源允许上限,应该可见这样的 Max open files            8096                 8096                 files
cat /proc/{pid_of_process}/limits
```

查看当前文件夹下的的最大size的10个文件夹, 并同时指定不搜索某个子文件夹 `./data`:
```
du -aBM --exclude=./data 2>/dev/null | sort -nr | head -n 10
```

修改docker的默认存储路径:
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
