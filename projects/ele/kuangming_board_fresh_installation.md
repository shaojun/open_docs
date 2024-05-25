# Frp client    
follow:    
https://gitee.com/bugslife/open_docs/blob/master/projects/edge/rv1126/prepare.md#install-frp-client    
to install.

# .NET CORE SDK
## download and install .NET CORE 3.1 SDK    
**In your PC**, manually open the link:    
https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/sdk-3.1.426-linux-arm32-binaries    
and copy the `direct link`, then input to **board shell** like:
```
mkdir -p /home/yunzao/Download
cd /home/yunzao/Download
wget https://download.visualstudio.microsoft.com/download/pr/2043e641-977d-43ac-b42a-f47fd9ee79ba/5b10d12a0626adaed720358ab8ad0b7e/dotnet-sdk-3.1.426-linux-arm.tar.gz
```
`unzip` and `set environment variables`:
```
mkdir -p $HOME/dotnet && tar zxf dotnet-sdk-3.1.426-linux-arm.tar.gz -C $HOME/dotnet
export DOTNET_ROOT=$HOME/dotnet
export PATH=$PATH:$HOME/dotnet
```
the environment varibles will lose when did a logoff or reboot, below will add them to automatically set when logon:
```
root@ebox:/home/yunzao/publish# nano ~/.bashrc
```
paste that 3 line into the **most below**:
```
export DOTNET_ROOT=$HOME/dotnet
export PATH=$PATH:$HOME/dotnet
export DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=1
```
then `ctrl+o` for save, and `ctrl+x` for quit.

## how to test
close current board shell, and open a new shell, input:
```
dotnet --info
```
should see meaningful messages.

# Board id file 
in **board shell**:
```
mkdir -p  /opt/nvidia/deepstream/deepstream-6.0/samples/configs/deepstream-app/
nano /opt/nvidia/deepstream/deepstream-6.0/samples/configs/deepstream-app/config_elenet.txt
```
in `nano` editing screen, input below default content:
```
[custom-uploader]
whoami=qua_board_default_board_id
```
then `ctrl+o` for save, and `ctrl+x` for quit.

# Object detection Application
ask developer to provide the application files, and then put into `/package/app/QuaAIDemo`:    
![image](https://github.com/shaojun/open_docs/assets/3241829/95589bd8-fa21-490c-9b6a-c729de2539fb)

restart the service:
```
systemctl restart quaai.service
```
## how to test
### by view log    
in **board shell**:
```
root@ebox:/package/app/QuaAIDemo# tail -30 log.txt
```
as you just restarted the service, should the last line of logs has the **recent time** and **content with success**.    
![image](https://github.com/shaojun/open_docs/assets/3241829/3fb2c363-5b8d-40cb-a20f-f68846c38da5)

### by view detection output    
in **board shell**:
```
root@ebox:/package/app/QuaAIDemo# python3 test_udp_heartbeat.py --events uploaded
```
this is for real time monitoring the detected objects upload to remote server, you should see **endless continues content** like:

![image](https://github.com/shaojun/open_docs/assets/3241829/acdd4688-4263-4d5c-b1bd-23a9cb19a56f)

each line indicates several `door_sign` objects were detected and get uploaded.
# LiteFccCore Application
## basic application files copy
in **board shell**:
```
mkdir -p /package/app/LiteFccCore/
```
**in your PC**, unzip `LiteFccCore.zip`, and upload the unzipped files into board under path `/package/app/LiteFccCore/`.    
the final should like:    
![image](https://github.com/shaojun/open_docs/assets/3241829/f43bf257-f736-4669-a45e-b3a697ec3256)

## kafka .so files copy
ask developer to provide kafka related file:    

![image](https://github.com/shaojun/open_docs/assets/3241829/c4872612-d480-40d1-bae1-5cf9f1251c8b)

copy above kafka related files to path: 
```
/lib/arm-linux-gnueabihf/
```
## create service
**in board shell** create the service file:
```
root@ebox:/package/app/LiteFccCore# nano /etc/systemd/system/LiteFccCore.service
```
input below content:
```
[Unit]
Description=LiteFccCore for controlling devices
Wants=network.target
After=nerwork.target ntp.service
[Service]
WorkingDirectory=/package/app/LiteFccCore
# every start of the service, include restart, will block 5 seconds
ExecStartPre=/bin/sleep 5

ExecStart=/root/dotnet/dotnet LiteFccCore.dll
Restart=always
# Restart service after 10 seconds if this service crashes:
RestartSec=10
SyslogIdentifier=LiteFccCore

[Install]
WantedBy=multi-user.target

```
then `ctrl+o` for save, and `ctrl+x` for quit.
Enable the service:
```
sudo systemctl enable LiteFccCore.service
sudo systemctl start LiteFccCore.service
```
## how to test
**in your PC**, make sure can LAN access to board by test `ping` in your PC terminal:
```
ping 192.168.177.1
```
**in your PC**, open browser with url: `http://192.168.177.1:8384/home/sensor`, should see web page.

# Watchdog service
ask developer to provide the latest `watchdog` files.  
in **board shell**:
```
mkdir -p /package/app/watch_dog/
```
**in your PC**, unzip `watch_dog.zip`, and upload the unzipped files into board under path `/package/app/watch_dog/`.  
in **board shell**:
```
cd /package/app/watch_dog
pip3 install requests
```
## create service
**in board shell** create the service file:
```
root@ebox:/package/app/watch_dog# nano /etc/systemd/system/watchdog.service
```
input below content:
```
[Unit]
Description=watch dog
Wants=network.target
After=network.target
[Service]
WorkingDirectory=/package/app/watch_dog/
# every start of the service, include restart, will block 5 seconds
# ExecStartPre=/bin/sleep 5

ExecStart=/usr/bin/python3 /package/app/watch_dog/watch_dog.py
Restart=always
# Restart service after 10 seconds if this service crashes:
RestartSec=10
SyslogIdentifier=watch_dog

[Install]
WantedBy=multi-user.target

```
then `ctrl+o` for save, and `ctrl+x` for quit.
Enable the service:
```
sudo systemctl enable watchdog.service
sudo systemctl start watchdog.service
```
