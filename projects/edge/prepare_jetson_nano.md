# Install SD card image for Jetson Nano

> Jetson Nano 2G and 4G has  **different**  image file, please confirm your device first

![输入图片说明](../../images/download_jetson_nano_sd_image_2_versions.png)

[Download the image file](https://developer.nvidia.com/embedded/downloads) and then [follow the steps](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-2gb-devkit#write) to write it to the SD card.


 **For simplify** , please create a `sudo` user with name: 
> eow    

password(left to your  _enter key_ ): 
> '

# Boost Jetson Nano to full power

> Note: For Jetson devices, use the following commands to manually increase the Jetson Power mode and maximize performance further by using the Jetson Clocks mode:
> 
> 
> ```
> sudo nvpmodel -m 0
> sudo /usr/bin/jetson_clocks
> ```
> 

# Speed up


```
export http_proxy='http://127.0.0.1:7890'
export https_proxy='https://127.0.0.1:7890'

unset http_proxy
unset https_proxy

wget https://bootstrap.pypa.io/pip/3.6/get-pip.py
python3 get-pip.py
python -m pip install --upgrade pip
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```


# Install the DeepStream 6

[Download](https://developer.nvidia.com/deepstream-6.0_6.0.0-1_arm64deb) the deepStream 6 package and then run below to install it:


`sudo apt-get install ./deepstream-6.0_6.0.0-1_arm64.deb`

# Install the Python App of DeepStream 6:

## 1 - Clone source 
**For project purpose** , clone this repo which contains extended functions:
 
```
cd /opt/nvidia/deepstream/deepstream/sources
sudo rm -rf deepstream_python_apps   #delete the old one
git clone https://github.com/shaojun/deepstream_python_apps
```

## 2 - Create python bindings

follow the below steps for short.

> Or you still can refer full steps at:
> https://github.com/NVIDIA-AI-IOT/deepstream_python_apps/blob/master/bindings/README.md

### 2.1 Base dependencies
- #### Ubuntu - 18.04 :
```
apt install -y git python-dev python3 python3-pip python3.6-dev python3.8-dev cmake g++ build-essential \
    libglib2.0-dev libglib2.0-dev-bin python-gi-dev libtool m4 autoconf automake
```

you may see the error in your  **jetson**  board like:



> The following packages have unmet dependencies:
>  python-gi-dev : Depends: python-gi (= 3.26.1-2) but 3.26.1-2ubuntu1 is to be installed
>                  Depends: python3-gi (= 3.26.1-2) but 3.26.1-2ubuntu1 is to be installed
> E: Unable to correct problems, you have held broken packages.

 **Solution 1**  is to purge current installed `python-gi` and `python3-gi`, this will wipe out some system build-in functions though I didn't see anything were broken:


```
sudo apt purge python-gi
sudo apt purge python3-gi
```

and the  **re-run**  the  **2.1 Base dependencies** to install the correct ones.

 **Solution 2** (have not tried yet) is to force install the specific version of packages that can let the process go through:


```
sudo apt-get install python-gi=3.26.1-2
sudo apt-get install python3-gi=3.26.1-2
```



- #### Ubuntu - 20.04 [use python-3.8, python-3.6 will not work] :
```
apt install python3-gi python3-dev python3-gst-1.0 python-gi-dev git python-dev \
    python3 python3-pip python3.8-dev cmake g++ build-essential libglib2.0-dev \
    libglib2.0-dev-bin python-gi-dev libtool m4 autoconf automake
```

### 2.2 Initialization of submodules
```
cd /opt/nvidia/deepstream/deepstream/sources/deepstream_python_apps
sudo git submodule update --init
```
### 2.3 Installing Gst-python
```
cd 3rdparty/gst-python/
sudo ./autogen.sh
sudo make
sudo make install
```

if you see errors like:


> Server certificate verification failed. CAfile: /etc/ssl/certs/ca-certificates.crt CRLfile: none

you can disable the GIT ssl certificate verification and run below the `make` by:

```
export GIT_SSL_NO_VERIFY=1
sudo -E sh ./autogen.sh
sudo make
sudo make install
```


## 3 - Building python bindings

Rather than build the bindings by yourself, here choose directly download it from [release page](https://github.com/NVIDIA-AI-IOT/deepstream_python_apps/releases).

Scroll to **Assets section**, for Jetson board,  choose the  **latest**  `.whl` with arch: `aarch64` to download.

![输入图片说明](../../images/download_ds_python_app_wheel_directly.png)


## 4 - Using python pip wheel

### 4.1 Installing the pip wheel

targeting the `whl` file you downloaded from above steps.

```
pip3 install ./pyds-1.1.0-py3-none*.whl
```
you may see errors like:

> Failed building wheel for pycairo

then check `4.1.1`.

#### 4.1.1 pip wheel troubleshooting
Please make sure you upgrade pip if the wheel installation fails
```
pip3 install --upgrade pip
```
and try `4.1` again.

### 4.2 Copy in external lib files:

for lib `librdkafka`:


```
sudo cp /opt/nvidia/deepstream/deepstream/sources/deepstream_python_apps/apps/deepstream-test51-on-test4/libs/librdkafka/* /opt/nvidia/deepstream/deepstream/lib/
sudo ldconfig 
```


## 5 - Testing

Enter apps root directory:

```
cd /opt/nvidia/deepstream/deepstream/sources/deepstream_python_apps/apps
```

### 5.1 launching test 1 app

```
cd deepstream-test1
python3 deepstream_test_1.py /opt/nvidia/deepstream/deepstream/samples/streams/sample_720p.h264
```

 start app may take **3 minutes or more**, then should see a popup video window with object detection.
 
### 5.2 launching test51 app

enter the directory of the app:

```
cd /opt/nvidia/deepstream/deepstream/sources/deepstream_python_apps/apps/deepstream-test51-on-test4
```

This app support detect objects from a rtsp input stream, and upload detected objects info to a remote kafka server.

as this app support rtsp input and output(for local monitoring), so install these packages is required:

```
sudo apt-get install libgstrtspserver-1.0-0 gstreamer1.0-rtsp
sudo apt-get install libgirepository1.0-dev
sudo apt-get install gobject-introspection gir1.2-gst-rtsp-server-1.0
```

> make sure you build the lib: `librdkafka` already which used for uploading to remote kafka server, otherwise refer [build kafka lib in Jetson](https://gitee.com/bugslife/open_docs/blob/master/projects/edge/kafka/kafka_dependency_on_Jetson.md).



#### 5.2.1 Edit the unique id: `whoami`

`whoami` is for identify each Jetson board device when multiple boards send messages to a remote kafka server,  **SHOULD**  keep this id unique  **per board**.

This `id` will be carried into a message and send to a remote  _kafka_  server as the objects detected constantly from local video stream, then the server message subscribers would know the source of the messages.

> this id also should be known by cloud side, so there must have a webpage provided by cloud side, like device registering portal, to allow you input the id there as well.

editing and input `whoamid` id:
```
ls config_elenet.txt  # you should see the file exists!
nano config_elenet.txt  # start edit it.
#input your unique id under the section custom-uploader -> whoami
``` 
can refer picture below, the red part is the `whoami` id:

![输入图片说明](../../images/edit_or_input_whoami_id_for_your_jetson_nano_board.png)

save above content by `ctr`+`o`, and copy config file to target path:

```
sudo cp config_elenet.txt /opt/nvidia/deepstream/deepstream/samples/configs/deepstream-app/
```

#### 5.2.2 Run

* Testing with a local video file

    the upload will against to default kafka server (url: `dev-iot.ipos.biz;9092`, topic: `test`):
    ```
    python3 deepstream_test_51.py -i file:///opt/nvidia/deepstream/deepstream/samples/streams/sample_720p.mp4 
    ```

* Testing with your own `pgie config file` and a local video file

    the upload will against to a specified kafka server:
    ```
    python3 deepstream_test_51.py -i file:///opt/nvidia/deepstream/deepstream/samples/streams/sample_720p.mp4 --pgie-config-file /opt/nvidia/deepstream/deepstream/samples/configs/tao_pretrained_models/config_infer_primary_trafficcamnet.txt --conn-str="dev-iot.ipos.biz;9092" --topic test_run 
    
    ```

* Testing with local video window(OSD) disabled, instead setup a RTSP server to output real-time object detection result:
    ```
    python3 deepstream_test_51.py -i file:///opt/nvidia/deepstream/deepstream/samples/streams/sample_720p.mp4 --pgie-config-file /opt/nvidia/deepstream/deepstream/samples/configs/tao_pretrained_models/config_infer_primary_trafficcamnet.txt --conn-str="dev-iot.ipos.biz;9092" --topic test_run --no-display    
    ```

    For observing the real-time object detection result, open a rtsp player to check, like [VLC](https://www.videolan.org/vlc/download-windows.html):
    
    ![输入图片说明](../../images/ds_python_app_output_rtsp_and_show_in_vlc.png)
    
    the rtsp url should like(replace the ip with the ip for your jetson board):
    
    > rtsp://192.168.0.126:8554/eow
    



* Testing with a camera RSTP stream with  **authentication**  required sample:

    no local display, and no output rtsp, this is mostly for production use:
    
    ```
    python3 deepstream_test_51.py -i rtsp://admin:KSglfmis1@36.153.41.21:2121 --no-display --no-output-rtsp    
    ```
    check the uploading result should be done at  _kafka_  server side.

## 6 - Regist app to system service and run it

for production, need install the `elenet deepstream python app` as a system service to facilitate maintenance.

### 6.1 Regist as a service

```
cd /etc/systemd/system 
sudo nano elenet.service
```
input below content:

> **make sure**  current system user account is: `eow` as the below content based on it, otherwise need update the below `User` to it;

> --no-display is required here, otherwise the app would not start.

> --no-output-rtsp depends on your situation, for production should keep it here for performance consideration.

```

[Unit]
Description=elenet deepstream python app for detect and upload.
Wants=network.target
After=network.target
[Service]
WorkingDirectory=/opt/nvidia/deepstream/deepstream-6.0/sources/deepstream_python_apps/apps/deepstream-test51-on-test4
# every start of the service, include restart, will block 5 seconds
# ExecStartPre=/bin/sleep 5

ExecStart=/usr/bin/python3 deepstream_test_51.py -i rtsp://admin:KSglfmis1@36.153.41.21:2121 --no-display --no-output-rtsp
Restart=always
# Restart service after 10 seconds if this service crashes:
RestartSec=10
SyslogIdentifier=elenet_ds_python_app
User=eow   
[Install]
WantedBy=multi-user.target
```

save above content by `ctr`+`o`.

### 6.2 Run the service

`Enable` service and it'll autoly get started in next boot:

`sudo systemctl enable elenet.service`

`Start` service right now:

`sudo systemctl start elenet.service`


> Check the status of the service:
> 
> `sudo systemctl status elenet.service`
> 
> if `elenet.service` file get updated, then reload the config and restart the service:
> 
> ```
> systemctl daemon-reload
> sudo systemctl restart elenet.service
> ```

### 6.3 Check service

check system process to see the service is running fine:

```
top
```

should see similar process is there with process name `python3`, and resource usage similar to below picture:

![输入图片说明](../../images/python_ds_app_run_with_processname_resourceused.png)



# Diagnose

## 安装TeamViewer到Jetson Nano

choose to download the 64-bit ARM version:
https://download.teamviewer.com/download/linux/teamviewer_arm64.deb

Run terminal command: 

```
sudo dpkg -i Downloads/teamviewer_15.23.9_arm64.deb
```

you’ll see some errors:
![输入图片说明](../../images/jetson_nano_install_tv_maysee_errorimage.png)

that’s fine, run terminal command: `sudo apt install -f` will fix it.

run terminal command to start the teamviewer as a service:

```
sudo systemctl enable teamviewerd.service
sudo service teamviewerd start
```

then, open the TeamViewer application via Internet->TeamViewer:

![输入图片说明](../../images/jetson_nano_install_tv_open_tv_setup_uiimage.png)


Please set a static password for TeamViewer via:

![输入图片说明](../../images/jetson_nano_setup_tv_static_passwordimage.png)


Also, check the below to allow start with system

![输入图片说明](../../images/jetson_nano_install_tv_enable_start_with_systemimage.png)


## Experienced memory low?
then could try:

- Disable teamviewer

This will freee up around ~300MB on **Jetson Nano 2G**
```
sudo systemctl stop teamviewerd.service
sudo systemctl disable teamviewerd.service
```

then check `top` to see all teamviewer related processes:  _TeamViewer_Desk_ ,  _teamviewerd_ ,  _TeamViewer_  are gone.


- Disabling the Desktop GUI

This will free up extra memory that the window manager and desktop uses (around ~40MB for `Xorg` on  **Jetson Nano 2G** , ~800MB for Unity/GNOME or ~250MB for LXDE)

You can disable the desktop temporarily, run commands in the console, and then re-start the desktop when you are done training:


```
$ sudo init 3     # stop the desktop
# log your user back into the console
# run the PyTorch training scripts
$ sudo init 5     # restart the desktop
```

If you wish to make this persistent across reboots, you can use the follow commands to change the boot-up behavior:


```
$ sudo systemctl set-default multi-user.target     # disable desktop on boot
$ sudo systemctl set-default graphical.target      # enable desktop on boot
```

Then after you reboot, the desktop will remain disabled or enabled (whichever default you set).
