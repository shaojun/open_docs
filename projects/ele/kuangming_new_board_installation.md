# .net core
## download and install 3.1 sdk    
manually open the link:    
https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/sdk-3.1.426-linux-arm32-binaries    
and copy the `direct link`, then input to shell like:
```
cd /home/yunzao/Download
wget https://download.visualstudio.microsoft.com/download/pr/2043e641-977d-43ac-b42a-f47fd9ee79ba/5b10d12a0626adaed720358ab8ad0b7e/dotnet-sdk-3.1.426-linux-arm.tar.gz
```
unzip and set environment varibles:
```
  mkdir -p $HOME/dotnet && tar zxf dotnet-sdk-3.1.426-linux-arm.tar.gz -C $HOME/dotnet
  export DOTNET_ROOT=$HOME/dotnet
  export PATH=$PATH:$HOME/dotnet
```
the environment varibles will lose when did a logoff or reboot, below will add them to automatically set when logon:
```
root@ebox:/home/yunzao/publish# nano ~/.bashrc
```
paste that 2 line into the most below:
```
export DOTNET_ROOT=$HOME/dotnet
export PATH=$PATH:$HOME/dotnet

```
disable the globalization:
```
export DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=1
```
## how to test
close current shell, and open a new shell, input:
```
dotnet --info
```
should see meaningful messages.
# ntp service
## install
```
 apt install ntp
```
## how to test
```
sudo systemctl status ntp
```
should see meaningful messages, and status is: `active (running)` 

# create board id file 
```
mkdir -p  /opt/nvidia/deepstream/deepstream-6.0/samples/configs/deepstream-app/
nano /opt/nvidia/deepstream/deepstream-6.0/samples/configs/deepstream-app/config_elenet.txt
```
in `nano` input below default content:
```
[custom-uploader]
whoami=qua_board_default_board_id
```
# install object detection application
ask developer to provide the latest `run_ele_qua.py` file.    
copy the file to path: `:/package/app/QuaAIDemo`    
edit the `startQuaAI.sh` to use `run_ele_qua.py`    
edit the `stopQuaAI.sh` to use `run_ele_qua.py`

# install the litefcccore application
ask developer to provide kafka related file.  
copy kafka related file to path: `/lib/arm-linux-gnueabihf/`    
ask developer to provide the latest `litefcccore` files.      
ask developer to provider file 'LiteFccCore.service'      
copy 'LiteFccCore.service' to path: `/etc/systemd/system`      
  ```
  sudo systemctl enable liteFccCore.service
  sudo systemctl start LiteFccCore.service
```
# install the watchdog service
ask developer to provide the latest `watchdog` files.  
