![image](https://github.com/user-attachments/assets/4d80040b-f4f2-4fcf-8f4f-292adfbca5e6)![image](https://github.com/user-attachments/assets/ee05c8cf-465b-44d4-8d5a-e3cc350cb1d3)# Install .NET CORE 6 SDK
* for Ubuntu 22.04
> the original steps: https://learn.microsoft.com/en-us/dotnet/core/install/linux-ubuntu-install?tabs=dotnet6&pivots=os-linux-ubuntu-2204
or just run:
```
sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-6.0
```
how to test?    
> run `dotnet --info` in terminal, should see meaningful text, example:
> 
> ![image](https://github.com/user-attachments/assets/3d011f48-bca6-42d0-be11-f77c69a1e156)


# Download LiteFccCore release
open url: https://github.com/shaojun/gogo-litefcccore/releases    
and choose the project and platform for you to download.
unzip and make sure the `release` folder is put under path `/home/dae/Downloads`

![image](https://github.com/user-attachments/assets/abb3bc78-c905-418a-984e-0aa281a093c4)

the folder strucutre should like:    
![image](https://github.com/user-attachments/assets/7bc29036-736e-41c0-b37e-ef10f858b618)

# Create LiteFccCore service for auto run
* create `.service` file
  create with `nano`:
  ```
  nano /etc/systemd/system/LiteFccCore.service
  ```
  sample:    
  ![image](https://github.com/user-attachments/assets/058cbbdb-170a-4402-9f1b-5021e974f82e)

* paste content to `.service` file
```
[Unit]
Description=LiteFccCore for controlling devices
Wants=network.target
After=nerwork.target ntp.service
[Service]
WorkingDirectory=/home/dae/Downloads/release/linux-x64/
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