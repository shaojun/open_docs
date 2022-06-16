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

# Setup Debian 10

## Install packages
putty into the Debian 10, and then install these packages:
```
sudo apt update
sudo apt install git
# clone with a branch
git clone https://gitlab.com/firefly-linux/external/rknn-toolkit.git -b rv1126_rv1109/firefly

#sudo apt install wget
#mkdir Download
#cd Download
#wget https://bootstrap.pypa.io/get-pip.py
#python3 get-pip.py

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
git clone https://github.com/shaojun/rv1126_elenet.git
cd rv1126_elenet
sudo python3 test_yolov5s_rtsp.py

# or for use specified rstp stream
# sudo python3 test_yolov5s_rtsp.py -i rtsp://YourSpecifiedUrl

# or for print debug to console and output infer result to local output folder:
# sudo python3 test_yolov5s_rtsp.py --enable-verbose true --enable-output true
```
## Config config_elenet.txt
`whoami` is for identify each board device when multiple boards send messages to a remote kafka server,  **SHOULD**  keep this id unique  **per board**.

This `id` will be carried into a message and send to a remote  _kafka_  server as the objects detected constantly from local video stream, then the server message subscribers would know the source of the messages.

> this id also should be known by cloud side, so there must have a webpage provided by cloud side, like device registering portal, to allow you input the id there as well.

editing and input `whoamid` id:
```
ls config_elenet.txt  # you should see the file exists!
nano config_elenet.txt  # start edit it.
#input your unique id under the section custom-uploader -> whoami
``` 
can refer picture below, the red part is the `whoami` id:

![输入图片说明](../../../images/edit_or_input_whoami_id_for_your_jetson_nano_board.png)

make sure the below path exists:
> /opt/nvidia/deepstream/deepstream/samples/configs/deepstream-app/

save above content by `ctr`+`o`, and copy config file to target path:

```
sudo cp config_elenet.txt /opt/nvidia/deepstream/deepstream/samples/configs/deepstream-app/
```

# Useful links
## how to install rknntookitlite
this is tested and works, basically the above steps are copied from this post
https://dev.t-firefly.com/thread-104204-1-1.html
## how to pack rootfs
https://dev.t-firefly.com/thread-118433-1-3.html
https://wiki.t-firefly.com/zh_CN/Firefly-RK3399/export_dev_sf.html