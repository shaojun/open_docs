# Flash Debian 10 into board

Install board Driver at your Windows PC from: 
> c:\xxxxx\DriverAssitant_v4.5\DriverInstall.exe

打开 RKDevTool at your Windows PC from：
> c:\xxxxx\RKDevTool\RKDevTool_Release_v2.86\RKDevTool.exe

拔掉板子上的 Usb Otg线、网线、电源线，然后确保板子上所有灯（网口旁边的两个绿色LED灯）都熄灭了。
再接入(请先不要接入电源线)Usb Otg线到板子的 **图示 USB TYPE A 接口** ，另一头到 PC:

![输入图片说明](../../../images/rv1126_firefly_jd4_how_to_conn_usb_otg_cable.jpg)


手按住板子上的 Recover 按钮, 保持住, 再接上电源线，等2秒钟，松 Recover 按钮，`RKDevTool`的主界面上将显示发现了 **LOADER设备**，如图：

![输入图片说明](../../../images/rv1126_firefly_jd4_rkdevtool_found_loader_device.png)

> 注意，进入 RecoveryMode 的板子是不会在 WindowsDeviceManager 里有任何item的。
> 在板子还未启动并进入 Debian 10 系统前， WindowsDeviceManager 也不会有任何设备出现，而只有板子在正常完全进入系统后，才有 rk3xxx 出现在usb设备中列表中

导入配置：

![输入图片说明](../../../images/RKDevTool_import_config.png)

Select the debian 10 config file:

![输入图片说明](../../../images/RKDevTool_select_debian10_config.png)

Select each `.img` one by one:

![输入图片说明](../../../images/RKDev_tool_flash_in_debian10_each_img.png)

点击**执行**, 开始刷入, 将持续10分钟左右，请**确保**看到 `RKDev_tool` 右侧进度都完成到`100%`:

![输入图片说明](../../../images/RKDevTool_flash_in_img_files.png)

板子随后将 **自动重启并进入** 新系统, 你现在可以 Unplug the USB-OTG cable from board. 

## how to test
确保主板上电, 请将一根`以太网网线`插入刚刷入完系统的板子的任意网口中。
>请确保此网线位于启用了`DHCP`的局域网中，以便板子可以自动获取到IP地址。

>板子所获取的具体 IP 地址只能通过人工登陆路由器管理页面，并查看`所分配的客户端IP列表`页面来获取得知.

>可能因具体项目(如 Smart Lift)的要求，板子将使用预先在镜像中固定的 IP 地址: 192.168.177.2 所以此时`DHCP`将不再有作用

测试最直接的办法是在位于同一局域网中的 PC 上，对板子进行 `ssh` (Windows上则应使用 [putty](https://the.earth.li/~sgtatham/putty/latest/w64/putty.exe)) 进行连接，看是否能连接成功。
另外的测试方法是使用串口来ssh连入主板，参考: [How to putty to board by serial port](https://gitee.com/bugslife/open_docs/blob/master/projects/edge/rv1126/prepare.md#how-to-putty-to-board-by-serial-port)

# Setup from official Debian 10

## Apt update
```
apt update
```
## Install Frp client
Download frp release (server and client are put together) from [frp release](https://github.com/fatedier/frp/releases/download/v0.43.0/frp_0.43.0_linux_arm.tar.gz) and untar it:
```
cd /home/firefly/Download/
tar -xzvf frp_0.43.0_linux_arm.tar.gz
# then the release files are in: /home/firefly/Download/frp_0.43.0_linux_arm/
```

As a frp client, edit the `/home/firefly/Download/frp_0.43.0_linux_arm/frpc.ini`，input below content and save:
```
[common]
server_addr = msg.glfiot.com
server_port = 7000
user = {{ .Envs.FRP_USER }}
[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = {{ .Envs.FRP_SSH_PORT }}

[camera]
type = tcp
local_ip = 192.168.177.3
local_port = 80
remote_port = {{ .Envs.FRP_CAMERA_PORT }}
```
Create a system service for auto start the `frp client` when system started:

```
sudo nano /etc/systemd/system/frpc.service
```

input below content(it defaultly use `6020` and `6021` port, you should gurantee it's **UNIQUE for all boards**, so please change it):

```
[Unit]
Description=Frp client
Wants=network.target
After=network.target
[Service]
Environment="FRP_USER=replace_me_with_real_board_id"
Environment="FRP_SSH_PORT=6020"
Environment="FRP_CAMERA_PORT=6021"
#before start the service, always sleep 5 second, for wait the system ready?
ExecStartPre=/bin/sleep 5
WorkingDirectory=/home/firefly/Download/frp_0.43.0_linux_arm/
ExecStart=/home/firefly/Download/frp_0.43.0_linux_arm/frpc -c 'frpc.ini'
Restart=always
#Restart service after 10 seconds if this service crashes:
RestartSec=10
[Install]
WantedBy=multi-user.target
```

`ctrl+o`, `y`, `ctrl+x`, save and exit from nano, then activate the service:
```
sudo systemctl enable frpc.service
sudo systemctl start frpc.service
# sudo systemctl status frpc.service
# sudo systemctl daemon-reload
```

then you can use it like:

![输入图片说明](../../../images/rv1126_connect_putty_via_frp.png)

Also, there's a frp build-in web service for check the overall connection (I installed it at msg.glfiot.com), the web page can be accessed at: http://msg.glfiot.com:7500/static/#/proxies/tcp, default username and password are:
>admin1
>
>admin1
## Install packages
Below use **serial port** to Putty into board, but the tcp putty should be similar.
Make sure the system is internet connected

### Install apt packages
```
sudo apt update
sudo apt-get install multiarch-support
sudo apt --fix-broken install
sudo apt install git
cd /home/firefly/

# Clone from official branch, clone the original repo is not used anymore since the huge size, then I picked 
# up the necessary files into https://github.com/shaojun/rv1126_elenet.git for convinient
# git clone https://gitlab.com/firefly-linux/external/rknn-toolkit.git -b rv1126_rv1109/firefly
#firefly@firefly:~/rknn-toolkit/rknn-toolkit-lite/rknn-toolkit-lite-v1.7.0.dev_0cfb22/requires$ sudo #dpkg -i libjasper1_1.900.1-debian1-2.4+deb8u6_armhf.deb
#firefly@firefly:~/rknn-toolkit/rknn-toolkit-lite/rknn-toolkit-lite-v1.7.0.dev_0cfb22/requires$ sudo #dpkg -i libjasper-dev_1.900.1-debian1-2.4+deb8u6_armhf.deb


git clone https://github.com/shaojun/rv1126_elenet.git
cd rv1126_elenet
sudo dpkg -i installation/requires/libjasper1_1.900.1-debian1-2.4+deb8u6_armhf.deb
sudo dpkg -i installation/requires/libjasper-dev_1.900.1-debian1-2.4+deb8u6_armhf.deb

sudo apt-get install libhdf5-dev
sudo apt-get install libatlas-base-dev
sudo apt-get install libqtgui4
sudo apt-get install libqt4-test

```
### Install Python packages
#### Install by simple-copy
the simple-copy could save some time compare to install by `pip`.
the `dist-packages` already prepared under repo `rv1126_elenet`, the content is like:
![输入图片说明](../../../images/python_dist_packages_content_structure.png)

```
chown -hR firefly:firefly /usr/local/lib/python3.7/dist-packages/
```
Copy the prepared `dist-packages` to board path:
```
cp -r /home/firefly/rv1126_elenet/installation/dist-packages/* /usr/local/lib/python3.7/dist-packages/
```
finally, the board folder structure is like:
![输入图片说明](../../../images/rv1126_debian10_copy_site_packages_directly.png)

you could then see if it works if no errors show after `import cv2`:
```
python3
import cv2
```
#### Install by pip - !!!don't do it if already did simple-copy
```
pip3 install numpy==1.16.3
pip3 install psutil==5.6.2
pip3 install ruamel.yaml==0.15.81
pip3 install bson
pip3 install kafka-python
pip3 install pymongo

#firefly@firefly:~/rknn-toolkit/rknn-toolkit-lite/rknn-toolkit-lite-v1.7.0.dev_0cfb22/requires$ pip3 #install opencv_python-4.0.1.24-cp37-cp37m-linux_armv7l.whl
pip3 install /home/firefly/rv1126_elenet/installation/requires/opencv_python-4.0.1.24-cp37-cp37m-linux_armv7l.whl

#firefly@firefly:~/rknn-toolkit/rknn-toolkit-lite/rknn-toolkit-lite-v1.7.0.dev_0cfb22/packages$ pip3 #install rknn_toolkit_lite-1.7.0.dev_0cfb22-cp37-cp37m-linux_armv7l.whl
pip3 install /home/firefly/rv1126_elenet/installation/packages/rknn_toolkit_lite-1.7.0.dev_0cfb22-cp37-cp37m-linux_armv7l.whl
```
## Prepare elenet

### Config  _whoami_ Id
`whoami` is for identify each board device when multiple boards send messages to a remote kafka server,  **SHOULD**  keep this id unique  **per board**.

This `id` will be carried into a message and send to a remote  _kafka_  server as the objects detected constantly from local video stream, then the server message subscribers would know the source of the messages.

> this id also should be known by cloud side, so there must have a webpage provided by cloud side, like device registering portal, to allow you input the id there as well.

### Editing and input `whoamid` id:
```
cd /home/firefly/rv1126_elenet
ls config_elenet.txt  # you should see the file exists!
nano config_elenet.txt  # start edit it.
#input your unique id under the section custom-uploader -> whoami
``` 
can refer picture below, the red part is the `whoami` id:

![输入图片说明](../../../images/edit_or_input_whoami_id_for_your_jetson_nano_board.png)

### Save and copy config_elenet.txt to:
make sure below path exists:
>  /opt/nvidia/deepstream/deepstream-6.0/samples/configs/deepstream-app/

or create it by: 
```
mkdir -p  /opt/nvidia/deepstream/deepstream-6.0/samples/configs/deepstream-app/
```


save above content in `nano` by `ctr`+`o`, and then copy config file to target path:

```
sudo cp config_elenet.txt  /opt/nvidia/deepstream/deepstream-6.0/samples/configs/deepstream-app/
```

### Run:

```
cd /home/firefly/rv1126_elenet
sudo python3 test_yolov5s_rtsp.py

# or for use specified rstp stream
# sudo python3 test_yolov5s_rtsp.py -i rtsp://YourSpecifiedUrl

# or for print debug info to console and output infer result to a local output folder:
# sudo python3 test_yolov5s_rtsp.py --enable-verbose true --enable-output true
```

### Run as service:

First create the `.service` file:

```
sudo nano /etc/systemd/system/elenet.service
```

input below content:

```
[Unit]
Description=elenet python app for detect and upload.
Wants=network.target
After=network.target
[Service]
WorkingDirectory=/home/firefly/rv1126_elenet
# every start of the service, include restart, will block 5 seconds
# ExecStartPre=/bin/sleep 5

ExecStart=/usr/bin/python3 test_yolov5s_rtsp.py
Restart=always
# Restart service after 10 seconds if this service crashes:
RestartSec=10
SyslogIdentifier=elenet_python_app
# User=firefly    comment out this line will defautly use root   
[Install]
WantedBy=multi-user.target
```

start the service:
```
sudo systemctl enable elenet.service
sudo systemctl start elenet.service
```

### Test if run correctly:

when running in `service mode` and defautly `enable-verbose` is disabled (you can enable it by `enable-verbose=true`), then you can try to receive a udp broadcast msg to know the status of the main app, the receive python script already embeded into main app, just run it by:

```
 cd /home/firefly/rv1126_elenet/
 python3 test_udp_heartbeat.py
```
Note, for performance consideration, the heartbeat fires for **every 5 times** of detection, sample like below:

>recv: b'{"sender": "192.168.177.9", "msg_type": "heartbeat", "total_infer_times": 500, "objects": "door_sign;door_sign;", >"timestamp": "2022-08-02 06:35:54.743971"}'
>recv: b'{"sender": "192.168.177.9", "msg_type": "heartbeat", "total_infer_times": 505, "objects": "door_sign;door_sign;", >"timestamp": "2022-08-02 06:35:57.718444"}'


# How to putty to board by serial port

> 根据经验，Windows机器对于此串口模块是**免驱**的，即只要连接`Micro Usb`头到串口模块上，同时`Usb-typeA`头到 PC 上，即可在系统中看到被识别出来：
> ![输入图片说明](../../../images/usb_to_ttl_connect_to_pc_have_a_com_port.png)
>
> 在确保线材的质量前提下，如果仍然未看到设备被识别，则请尝试安装 Firefly USB转UART串口模块[驱动程序](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers).

wiring like below picture:
![输入图片说明](../../../images/putty_to_firefly_rv1126_by_serial.jpg)

you should see a com port is created on you Windows PC:
![输入图片说明](../../../images/usb_to_ttl_connect_to_pc_have_a_com_port.png)

open putty, Serial line input `COM3` for my situation,Speed to `1500000`,

![输入图片说明](../../../images/open_putty_to_firefly_rv1126_board_with_serial.png)

`Open` then you should get the terminal console, after several `Enter`, input `/sbin/ifconfig` to check ip address (make sure you have LAN cable connected and with `dhcp` enabled):
```
root@firefly:/home/firefly# /sbin/ifconfig
enx00e04cc4edfa: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 00:e0:4c:c4:ed:fa  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.130  netmask 255.255.255.0  broadcast 192.168.0.255
        ether ba:c9:11:2f:02:2c  txqueuelen 1000  (Ethernet)
        RX packets 10752016  bytes 685211272 (653.4 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1417689  bytes 374737499 (357.3 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 63
```
the `eth0` is the LAN interface, the ip address for my situation is `192.168.0.130`, you can then also use the ip to putty in.

# Useful links
## create frp server service

```
[Unit]
Description=Frp server
Wants=network.target
After=network.target
[Service]
#before start the service, always sleep 5 second, for wait the system ready?
ExecStartPre=/bin/sleep 5
WorkingDirectory=/home/Downloads/frp_0.43.0_linux_amd64/
ExecStart=/home/Downloads/frp_0.43.0_linux_amd64/frps -c 'frps.ini'
Restart=always
#Restart service after 10 seconds if this service crashes:
RestartSec=10
[Install]
WantedBy=multi-user.target
```
## how to install rknntookitlite
this is tested and works, basically the above steps is highly concluded from this post

https://dev.t-firefly.com/thread-104204-1-1.html
## how to pack rootfs
### raw post:
https://dev.t-firefly.com/thread-118433-1-3.html

https://wiki.t-firefly.com/zh_CN/Firefly-RK3399/export_dev_sf.html

https://wiki.t-firefly.com/zh_CN/CORE-1126-JD4/Debian10.html#
https://wiki.t-firefly.com/zh_CN/CORE-1126-JD4/Debian10.html#fen-qu-jie-shao
### steps
* Prepare a 32G (or above size) USB Drive
* At your windows PC, download the _DiskGenius_ tool
https://engdownload.eassos.cn/DGEngSetup5431342.exe

    Format the USB Drive with `ext4` by _DiskGenius_:

    ![输入图片说明](../../../images/use_disk_genius_to_format_usb_drive_with_ext4.png)

* Copy the export tool to board Debian system
The export tool is at `resource/ff_export_rootfs_buildroot.tar`.
Copy it to `firefly@firefly:~/Download/`.
De-compress it by: `tar -xzvf ff_export_rootfs_buildroot.tar`

* Plug in the USB Drive to board's usb port

    Check Usb Drive name in board Debian
    ```
    firefly@firefly:~/Download/ff_export_rootfs_buildroot$ lsblk
    NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sda            8:0    1 58.7G  0 disk
    `-sda1         8:1    1 58.7G  0 part
    mmcblk0      179:0    0 14.6G  0 disk
    |-mmcblk0p1  179:1    0    4M  0 part
    |-mmcblk0p2  179:2    0    4M  0 part
    |-mmcblk0p3  179:3    0   32M  0 part
    |-mmcblk0p4  179:4    0   32M  0 part
    |-mmcblk0p5  179:5    0   32M  0 part
    |-mmcblk0p6  179:6    0    2G  0 part /root-ro
    |-mmcblk0p7  179:7    0  192M  0 part
    `-mmcblk0p8  179:8    0 12.3G  0 part /userdata
    mmcblk0boot0 179:32   0    4M  1 disk
    mmcblk0boot1 179:64   0    4M  1 disk
    ```
    that `sda1` with size 58.7G is your USB Drive.
* Create a folder in board Debian for mount Usb Drive
`mkdir /media/usb_drive`

* Mount Usb Drive in board Debian
    `sudo mount /dev/sda1 /media/usb_drive`
    should see:
    >[   93.190391] EXT4-fs (sda1): mounted filesystem with ordered data mode. Opts: (null)

* Export
    in board Debian, from experience, it'll take 15m to go, if you see much longer than this, better change another USB Drive:
    ```
    apt install rsync
    firefly@firefly:~/Download/ff_export_rootfs$ sudo ./ff_export_rootfs /media/usb_drive/
    ```
    if you see folder not found error like:
   
    >Could not open /media/usb_drive/Firefly_Debian_GNU/Linux_10_(buster)_ext4_202209090708.img: No such file or directory

    then create the folder with: `mkdir /media/usb_drive/Firefly_Debian_GNU/`

    if everything is good, you should see below, and finally the file will be created, the whole process should be done in 10m:
    ```
    MEDIA FREE SPACE SIZE    55975   MBytes
    EXPORT IMAGE SIZE        2404    MBytes
    BLOCK_COUNT 2408132
    INODE_COUNT 182183
    [  160.791718] EXT4-fs (loop0): mounted filesystem with ordered data mode. Opts: (null)
    sync...


    sync finish
    e2fsck 1.44.5 (15-Dec-2018)
    Export rootfs to /media/usb_drive/Firefly_Debian_GNU/Linux_10_(buster)_ext4_202209090709.img Success

    ```

    
    see the export file sample:
    ![输入图片说明](../../../images/rootfs_export_to_img_file_under_usb_drive_mount_folder.png)
* Resize
    in board Debian
    ```
    cd /media/usb_drive/Firefly_Debian_GNU
    firefly@firefly:/media/usb_drive/Firefly_Debian_GNU$ sudo /sbin/e2fsck -p -f Firefly_ext4_202206170232.img
    rootfs: 48731/172520 files (14.6% non-contiguous), 7706484/9290489 blocks
    
    firefly@firefly:/media/usb_drive/Firefly_Debian_GNU$ sudo /sbin/resize2fs -M Firefly_ext4_202206170232.img
    resize2fs 1.44.5 (15-Dec-2018)
    Resizing the filesystem on Firefly_ext4_202206170232.img to 7702411 (1k) blocks.
    The filesystem on Firefly_ext4_202206170232.img is now 7702411 (1k) blocks long.
    ```

* Repack - **!!!SKIP this for RV1126**
**YOU SHOULD NOT DO THIS FOR RV1126**
COPY `resource/firefly-rk3399-linux-repack.tgz` to your **Ubuntu PC**.
COPY the new packed and resized `.img` file (like above resized one: _Firefly_ext4_202206170232.img_) to same folder, and rename it to `udpate.img`, then the folder structure is like:
    ```
    (base) shawn@DESKTOP-9NG0VFK:~/Downloads/firefly-rk3399-linux-repack$ ls
    bin  output  pack.sh  Readme.md  unpack.sh  update.img
    ```
    start unpack:
    ```
    sudo ./unpack.sh
    start to unpack update.img...
    ********RKImageMaker ver 1.66********
    Unpacking image, please wait...
    Error:Check update.img failed!
    Press any key to quit:
    ```

* Prepare the `parameter.txt` file
here take an export file: `Firefly_ext4_202206240234.img` as example (not the one in above sample, but the theory should be the same).
Copy the exported file `Firefly_ext4_202206240234.img` from board to Your Windows PC:
![输入图片说明](../../../images/copy_rv1126_export_rootfs_to_windows_pc.png)
Can see its size is 5.84GB.
`parameter.txt` is used in firmware flashing for locating each partition (you can see several `*.img` files are used in flashing), here is the sample of the `parameter.txt` from firefly official Debian 10 firmware packages:

    >FIRMWARE_VER: 8.1
MACHINE_MODEL: RV1126
MACHINE_ID: 007
MANUFACTURER: RV1126
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00010000@0x00008000(boot),0x00010000@0x00018000(recovery),0x00010000@0x00028000(backup),0x00C00000@0x00038000(rootfs),0x00060000@0x00C38000(oem),-@0x00C98000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9

    As you're packing your own fireware, then only 2 Partition info need to be updated correspondingly with your new exported file(size), they are in the: 
    >0x00C00000@0x00038000(rootfs),0x00060000@0x00C38000(oem),-@0x00C98000(userdata:grow)

    understand as: **Partition_Size@Partition_Start_Address**

    * Partition `rootfs`
    the exported file is actually a new `rootfs` that is supposed to replacing the old (official) one, once replaced, the partition size and follwed partitions StartAddress are needed to adjust as well.
    `0x00C00000` is `rootfs` partition size, it must greater than the file `rootfs.img` size, the requried `rootfs` partition size's caculation can be understood with: `0x00C00000  块 * 512 字节每块 / 1024 / 1024 = 6144 MByte`, obvious the `6144` is greater(**and must**) than actual `rootfs.img` file size(5.84GB), it's a proper value here, it also means you can set higher value.

    * Partition `oem`:
        >0x00060000@0x00C38000(oem),-@0x00C98000(userdata:grow)

        because these 2 partitions are following `rootfs` partition, so with the adjustment of `rootfs` partition, we need to update above 2 partitions' start address as well with rule: `分区大小 + 所在地址 = 下一个分区的所在地址` which is `0x00C00000 + 0x00038000 = 0x00C38000`

* Flashing `*.img` files to board
for your own fireware pack, replace that 2 files with yours, others still use the firefly official ones.
boot your board to flashing mode with previous steps.

import:

![输入图片说明](../../../images/rv_Dev_tool_import_config_file.png)

select file:

![输入图片说明](../../../rv_dev_tool_select_debian10_configfile.png)

input each files:

![输入图片说明](../../../images/flashing_the_rebuild_rootfs_and_parameter_to_board.png)

 Detail steps please refer section: *Flash in Debian 10 to board*

