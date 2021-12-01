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
git submodule update --init
```
### 2.3 Installing Gst-python
```
cd 3rdparty/gst-python/
./autogen.sh
make
make install
```
### 2.4 - Deepstream SDK

Go to https://developer.nvidia.com/deepstream-sdk, download and install Deepstream SDK and its dependencies

## 3 - Building the bindings

here directly download it from [release page](https://github.com/NVIDIA-AI-IOT/deepstream_python_apps/releases)

for Jetson board, from the **Assets** section, choose the latest `.whl` with arch: `aarch64`, download link sample like: https://github.com/NVIDIA-AI-IOT/deepstream_python_apps/releases/download/v1.1.0/pyds-1.1.0-py3-none-linux_aarch64.whl

...
...
...

## 4 - Using the generated pip wheel

### 4.1 Installing the pip wheel

targeting the `whl` file you downloaded from above steps.

```
pip3 install ./pyds-1.1.0-py3-none*.whl
```
you may see errors, then check `4.1.1`
#### 4.1.1 pip wheel troubleshooting
Please make sure you upgrade pip if the wheel installation fails
```
pip3 install --upgrade pip
```
and try `4.1` again.
### 4.2 launching test 1 app
```
cd apps/deepstream-test1
python3 deepstream_test_1.py <input .h264 file>
```
