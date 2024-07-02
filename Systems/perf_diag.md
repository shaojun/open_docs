view realtime threads' stack:
```
dotnet-stack report --process-id $(pidof dotnet)
```
using `dotnet-trace` to keep collecting all threads stack for a given process(update the process name of `dotnet` to yours):
```
root@iot-web-4001:~/shao# ./dotnet-trace collect --process-id $(pidof dotnet)
```
say keep collecting 60 seconds, and then stop.    
now display the top n threads that using most time:
```
root@iot-web-4001:~/shao# ./dotnet-trace report dotnet_20240702_144736.nettrace topN -n 40 -v
```
