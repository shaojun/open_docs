## view current all threads stack:
```
dotnet-stack report --process-id $(pidof dotnet)
```
or output to a file:
```
dotnet-stack report --process-id $(pidof dotnet) | your_file.txt
```
##  keep collecting all threads stack for a given process and a period of time:
```
root@iot-web-4001:~/shao# ./dotnet-trace collect --process-id $(pidof dotnet)
```
say keep collecting 60 seconds, and then stop.    
now display the top n threads that used most CPU time:
```
root@iot-web-4001:~/shao# ./dotnet-trace report dotnet_20240702_144736.nettrace topN -n 40 -v
```
## Displays periodically refreshing values of AspNetCore.Hosting counters
```
dotnet-counters monitor --process-id $(pidof dotnet) --counters Microsoft.AspNetCore.Hosting 
```
sample:
> Press p to pause, r to resume, q to quit.    
>    Status: Running    
>     
> [Microsoft.AspNetCore.Hosting]    
>     Current Requests                                                       8    
>     Failed Requests                                                        0    
>     Request Rate (Count / 1 sec)                                         167    
>     Total Requests                                                     7,796    

## Collect all counters at a refresh interval of 3 seconds and generate a csv as output
```
dotnet-counters collect --process-id $(pidof dotnet) --refresh-interval 3 --format csv
```
