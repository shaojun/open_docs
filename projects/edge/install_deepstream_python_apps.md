# Install SD card image for Jetson Nano

> Jetson Nano 2G and 4G has different image file, please confirm your device first

![输入图片说明](download_jetson_nano_sd_image_2_versions.png)
[download](https://developer.nvidia.com/embedded/downloads)

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



# Install the DeepStream 6

[Download](https://developer.nvidia.com/deepstream-6.0_6.0.0-1_arm64deb) the deepStream 6 package and then run below to install it:


`sudo apt-get install ./deepstream-6.0_6.0.0-1_arm64.deb`

# Install the Python App of DeepStream 6:

## 1 Clone source 
**For project purpose** , clone this repo which contains extended functions:
 
```
cd /opt/nvidia/deepstream/deepstream/sources
git clone https://github.com/shaojun/deepstream_python_apps
```

## 2 Create python bindings
Can refer full doc at [deepstream_python_apps bindings](https://github.com/NVIDIA-AI-IOT/deepstream_python_apps/blob/master/bindings/README.md)
or follow this for short:
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
git submodule update --init
```
### 2.3 Installing Gst-python
```
cd 3rdparty/gst-python/
./autogen.sh
make
make install
```

if you see errors like:


> Server certificate verification failed. CAfile: /etc/ssl/certs/ca-certificates.crt CRLfile: none

you can disable the GIT ssl certificate verification and re-run the make by:

```
export GIT_SSL_NO_VERIFY=1
sudo -E sh ./autogen.sh
sudo make
sudo install
```


### 3 - Building the bindings

Rather than build the bindings by yourself, here choose directly download it from [release page](https://github.com/NVIDIA-AI-IOT/deepstream_python_apps/releases).

Scroll to **Assets section**, for Jetson board,  choose the  **latest**  `.whl` with arch: `aarch64` to download.

![输入图片说明](download_ds_python_app_wheel_directly.png)


## 4 - Using the generated pip wheel

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
### 4.2 launching test 1 app
```
cd /opt/nvidia/deepstream/deepstream/sources/deepstream_python_apps/apps/deepstream-test1
python3 deepstream_test_1.py /opt/nvidia/deepstream/deepstream/samples/streams/sample_720p.h264
```
### 4.3 launching test 4 app
make sure you build the `librdkafka` already by checking if the below file exists:
```
/opt/nvidia/deepstream/deepstream/lib/libnvds_kafka_proto.so
```

If not, refer [build kafka lib in Jetson](https://gitee.com/bugslife/open_docs/blob/master/projects/edge/kafka/kafka_dependency_on_Jetson.md).

Run:
```
cd apps/deepstream-test4
python3 deepstream_test_4.py -i /opt/nvidia/deepstream/deepstream/samples/streams/sample_720p.h264 -p /opt/nvidia/deepstream/deepstream/lib/libnvds_kafka_proto.so --conn-str="dev-iot.ipos.biz;9092"  --topic="test" -s 0
```

### 4.4 prepare the folder structure for owned trained tao models

for deploy custom models trained from `TAO`, suggest follow the step to create folder structures for better orgnize, you're  **not required**  to use the naming of `doorsignnet`, just use your own but don't forget  **update the PGIE file** :


```
cd /opt/nvidia/deepstream/deepstream/samples/models/
mkdir tao_pretrained_models/doorsignnet
#copy your models and labels files here, see below picture for refer:

```
![输入图片说明](../../images/deepstream_copy_model_to_tao_folder_sample.png)

also put the  **PGIE file**  here:

![输入图片说明](../../images/deepstream_pgie_from_tao_sample.png)


### 4.5 launching test51 app

as this app support rtsp input and output, so install these packages:

```
sudo apt-get install libgstrtspserver-1.0-0 gstreamer1.0-rtsp
sudo apt-get install libgirepository1.0-dev
sudo apt-get install gobject-introspection gir1.2-gst-rtsp-server-1.0
```


```
cd /opt/nvidia/deepstream/deepstream/sources/deepstream_python_apps/apps/deepstream-test51-on-test4
```
Input/Edit the unique id: `whoami` for identify the Jetson device you're using, this `id` will be carried and send from current app to remote kafka server:
```
nano cfg_kafka.txt
#input your unique id under the section custom-uploader -> whoami
``` 
For using default `pgie config file` ( _dstest51_pgie_config.txt_ ) and a local video file, then upload to default kafka server (url: `dev-iot.ipos.biz;9092`, topic: `test`):

```
python3 deepstream_test_51.py -i file:///opt/nvidia/deepstream/deepstream/samples/streams/sample_720p.mp4 
```

For using your own `pgie config file` and a local video file, then upload to specified kafka server:
```
python3 deepstream_test_51.py -i file:///home/eow/Downloads/video_sample_from_screen_record/screen_captured_elemotor_3person_2111241020.mp4 --pgie-config-file /opt/nvidia/deepstream/deepstream/samples/configs/tao_pretrained_models/config_infer_primary_trafficcamnet.txt --conn-str="dev-iot.ipos.biz;9092" --topic test 

```

For disable local video window, instead setup a RTSP server:
```
python3 deepstream_test_51.py -i file:///home/eow/Downloads/video_sample_from_screen_record/screen_captured_elemotor_3person_2111241020.mp4 --pgie-config-file /opt/nvidia/deepstream/deepstream/samples/configs/tao_pretrained_models/config_infer_primary_trafficcamnet.txt --conn-str="dev-iot.ipos.biz;9092" --topic test --no-display

```

Read a remote RSTP stream with  **authentication**  requried sample:
```
python3 deepstream_test_51.py -i rtsp://admin:KSglfmis1@36.153.41.21:2121 --pgie-config-file /opt/nvidia/deepstream/deepstream/samples/configs/tao_pretrained_models/config_infer_primary_doorsignnet.txt 

```




# Experienced memory low?
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
