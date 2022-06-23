# Flash Debian 10

>进入 RecoveryMode 的板子是不会在 WindowsDeviceManager 里有任何item的。

拔掉 usb otg 线，拔掉电源，确保板子上所有灯都熄灭了，再接入usb otg线到PC。
打开 RKDevTool：
> c:\xxxxx\RKDevTool\RKDevTool_Release_v2.86\RKDevTool.exe

手按住板子上的 Recover 按钮, 保持, 再接上电源，等2秒钟，松 Recover 按钮，RKDevTool应该能发现adb device. 
> 在现在情况下, WindowsDeviceManager不会有任何设备出现，而只有板子在正常完全进入系统后，才有rk3xxx出现在usb设备中

![输入图片说明](../../../images/RKDevTool_import_config.png)

Select the debian 10 config file:

![输入图片说明](../../../images/RKDevTool_select_debian10_config.png)

Select each `.img` one by one:
![输入图片说明](../../../images/RKDev_tool_flash_in_debian10_each_img.png)

点击“执行”, 开始刷入：

![输入图片说明](../../../images/RKDevTool_flash_in_img_files.png)

## how to adb
For Ubuntu PC, unplug otg usb, unplug power cable, all board LED off, then plug power cable, wait 5s, plug in otg usb, should see a mobile icon in PC, and then `adb devices` should see the board.

## windows
never worked.
# Setup Debian 10

## Install packages
putty into the Debian 10, make sure the system is internet connected,ap and then install these packages:
```
sudo apt update
sudo apt install git
cd /home/firefly/
# clone with a branch
git clone https://gitlab.com/firefly-linux/external/rknn-toolkit.git -b rv1126_rv1109/firefly

#sudo apt install wget
#mkdir Download
#cd Download
#wget https://bootstrap.pypa.io/get-pip.py
#python3 get-pip.py


chown -hR firefly:firefly /usr/local/lib/python3.7/dist-packages/
![输入图片说明](../../../images/rv1126_debian10_copy_site_packages_directly.png)


#建议全都pip3安装?no need python3 -m
sudo apt-get install python3-pip
python3 -m pip install numpy==1.16.3
python3 -m pip install psutil==5.6.2
python3 -m pip install ruamel.yaml==0.15.81
pip3 install bson
pip3 install kafka-python
pip3 install pymongo

sudo apt-get install multiarch-support
sudo apt --fix-broken install
firefly@firefly:~/rknn-toolkit/rknn-toolkit-lite/rknn-toolkit-lite-v1.7.0.dev_0cfb22/requires$ sudo dpkg -i libjasper1_1.900.1-debian1-2.4+deb8u6_armhf.deb
firefly@firefly:~/rknn-toolkit/rknn-toolkit-lite/rknn-toolkit-lite-v1.7.0.dev_0cfb22/requires$ sudo dpkg -i libjasper-dev_1.900.1-debian1-2.4+deb8u6_armhf.deb
sudo apt-get install libhdf5-dev
sudo apt-get install libatlas-base-dev
sudo apt-get install libqtgui4
sudo apt-get install libqt4-test
firefly@firefly:~/rknn-toolkit/rknn-toolkit-lite/rknn-toolkit-lite-v1.7.0.dev_0cfb22/requires$ python3 -m pip install opencv_python-4.0.1.24-cp37-cp37m-linux_armv7l.whl
# notice the cv2 installed path, if it's like this path:/home/firefly/.local/lib/python3.7/site-packages
then need add this at the most begining of application python file, otherwise, running the python will got 'cannot found the cv2' error
#import sys
#sys.path.append('/home/firefly/.local/lib/python3.7/site-packages')

firefly@firefly:~/rknn-toolkit/rknn-toolkit-lite/rknn-toolkit-lite-v1.7.0.dev_0cfb22/packages$ python3 -m pip install rknn_toolkit_lite-1.7.0.dev_0cfb22-cp37-cp37m-linux_armv7l.whl
firefly@firefly:~/rknn-toolkit/rknn-toolkit-lite/rknn-toolkit-lite-v1.7.0.dev_0cfb22/examples-lite/inference_with_lite$ sudo python3 test.py
```
## Install elenet
```
cd ~/
cd
cd rv1126_elenet
```
## Config _whoami_ Id
`whoami` is for identify each board device when multiple boards send messages to a remote kafka server,  **SHOULD**  keep this id unique  **per board**.

This `id` will be carried into a message and send to a remote  _kafka_  server as the objects detected constantly from local video stream, then the server message subscribers would know the source of the messages.

> this id also should be known by cloud side, so there must have a webpage provided by cloud side, like device registering portal, to allow you input the id there as well.

### Editing and input `whoamid` id:
```
ls config_elenet.txt  # you should see the file exists!
nano config_elenet.txt  # start edit it.
#input your unique id under the section custom-uploader -> whoami
``` 
can refer picture below, the red part is the `whoami` id:

![输入图片说明](../../../images/edit_or_input_whoami_id_for_your_jetson_nano_board.png)

### Save and copy to:
make sure below path exists:
>  /opt/nvidia/deepstream/deepstream-6.0/samples/configs/deepstream-app/
can create it by `mkdir -p  /opt/nvidia/deepstream/deepstream-6.0/samples/configs/deepstream-app/`


save above content by `ctr`+`o`, and copy config file to target path:

```
sudo cp config_elenet.txt  /opt/nvidia/deepstream/deepstream-6.0/samples/configs/deepstream-app/
```

run application:

```
sudo python3 test_yolov5s_rtsp.py

# or for use specified rstp stream
# sudo python3 test_yolov5s_rtsp.py -i rtsp://YourSpecifiedUrl

# or for print debug to console and output infer result to local output folder:
# sudo python3 test_yolov5s_rtsp.py --enable-verbose true --enable-output true
```
# Useful links
## how to install rknntookitlite
this is tested and works, basically the above steps is highly concluded from this post

https://dev.t-firefly.com/thread-104204-1-1.html
## how to pack rootfs
### raw post:
https://dev.t-firefly.com/thread-118433-1-3.html

https://wiki.t-firefly.com/zh_CN/Firefly-RK3399/export_dev_sf.html

### steps
* Prepare a 32G (or above size) Usb Drive
* At your windows PC, download the _DiskGenius_ tool
https://engdownload.eassos.cn/DGEngSetup5431342.exe
* Format the usb drive with `ext4` by tool
![输入图片说明](../../../images/use_disk_genius_to_format_usb_drive_with_ext4.png)
* Copy the export tool to board Debian system
The export tool is at `resource/ff_export_rootfs_buildroot.tar`.
Copy it to `firefly@firefly:~/Download/`.
De-compress it by: `tar -xzvf ff_export_rootfs_buildroot.tar`
* Plug in the Usb Drive to board usb port
* Check Usb Drive name in Debian
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
* Create a folder for mount Usb Drive
`mkdir /media/usb_drive`
* Mount Usb Drive
`sudo mount /dev/sda1 /media/usb_drive`
* Export
    will take minutes to go:
    ```
    firefly@firefly:~/Download/ff_export_rootfs_buildroot$ sudo ./ff_export_rootfs /media/usb_drive/
    MEDIA FREE SPACE SIZE    55975   MBytes
    EXPORT IMAGE SIZE        8568    MBytes
    find: '/proc/896/task/896/net': Invalid argument
    find: '/proc/896/net': Invalid argument
    find: '/proc/936/task/936/net': Invalid argument
    find: '/proc/936/net': Invalid argument
    find: '/proc/1139/task/1139/net': Invalid argument
    find: '/proc/1139/net': Invalid argument
    BLOCK_COUNT 9290489
    INODE_COUNT 169877
    sync...
    sync finish
    Export rootfs to /media/usb_drive//Firefly_ext4_202206170232.img Success
    ```
    see the export file sample:
    ![输入图片说明](../../../images/rootfs_export_to_img_file_under_usb_drive_mount_folder.png)
* Resize
    ```
    firefly@firefly:/media/usb_drive$ sudo /sbin/e2fsck -p -f Firefly_ext4_202206170232.img
    [sudo] password for firefly:
    rootfs: 48731/172520 files (14.6% non-contiguous), 7706484/9290489 blocks
    firefly@firefly:/media/usb_drive$ sudo /sbin/resize2fs -M Firefly_ext4_202206170232.img
    resize2fs 1.44.5 (15-Dec-2018)
    Resizing the filesystem on Firefly_ext4_202206170232.img to 7702411 (1k) blocks.
    The filesystem on Firefly_ext4_202206170232.img is now 7702411 (1k) blocks long.

    ```

* repack
COPY resource/firefly-rk3399-linux-repack.tgz to your **Ubuntu PC**.
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