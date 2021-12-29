

> Note: For Jetson devices, use the following commands to manually increase the Jetson Power mode and maximize performance further by using the Jetson Clocks mode:
> 
> 
> ```
> sudo nvpmodel -m 0
> sudo /usr/bin/jetson_clocks
> ```
> 

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
Ubuntu - 18.04 :
```
apt install -y git python-dev python3 python3-pip python3.6-dev python3.8-dev cmake g++ build-essential \
    libglib2.0-dev libglib2.0-dev-bin python-gi-dev libtool m4 autoconf automake
```

Ubuntu - 20.04 [use python-3.8, python-3.6 will not work] :
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

### 2.4 - DeepStream SDK

Go to https://developer.nvidia.com/deepstream-sdk, download and install DeepStream SDK and its dependencies

### 3 - Building the bindings

Rather than build the bindings by yourself, here choose directly download it from [release page](https://github.com/NVIDIA-AI-IOT/deepstream_python_apps/releases), scroll to **Assets section**, for Jetson board,  choose the  **latest ** `.whl` with arch: `aarch64` to download.

download link sample like,  **may not the latest** : 

https://github.com/NVIDIA-AI-IOT/deepstream_python_apps/releases/download/v1.1.0/pyds-1.1.0-py3-none-linux_aarch64.whl

...

...

...


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
cd apps/deepstream-test1
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

### 4.4 launching test51 app

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
python3 deepstream_test_51.py -i file:///home/eow/Downloads/video_sample_from_screen_record/screen_captured_elemotor_3person_2111241020.mp4 --pgie-config-file /opt/nvidia/deepstream/deepstream/samples/configs/tao_pretrained_models/config_infer_primary_trafficcamnet.txt --conn-str dev-iot.ipos.biz;9092 --topic test 

```

For disable local video window, instead setup a RTSP server:
```
python3 deepstream_test_51.py -i file:///home/eow/Downloads/video_sample_from_screen_record/screen_captured_elemotor_3person_2111241020.mp4 --pgie-config-file /opt/nvidia/deepstream/deepstream/samples/configs/tao_pretrained_models/config_infer_primary_trafficcamnet.txt --conn-str dev-iot.ipos.biz;9092 --topic test --no-display

```