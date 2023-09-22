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
查看和更改某进程被设定的资源上限:
```
#查看
cat /proc/{pid_of_process}/limits
#更改某个线程的 nofile 限制上限到 4096:
prlimit -n4096 -p pid_of_process
```

