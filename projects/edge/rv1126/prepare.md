进入recovery mode的板子是不会在windows下的device manager里有任何item的。
拔掉usb otg线，拔掉电源，再接入usb otg到pc, 打开dev tool, 手按住update button, 保持， 再接上电源，等2秒钟，松按钮， dev tool应该能发现adb device. 在现在情况下， windows device manager不会有任何设备出现，而只有在正常进入系统，才有rk3xxx

flash debian 10
putty in
sudo apt update
sudo apt install git
git clone https://gitlab.com/firefly-linux/external/rknn-toolkit.git -b rv1126_rv1109/firefly

#sudo apt install wget
#mkdir Download
#cd Download
#wget https://bootstrap.pypa.io/get-pip.py
#python3 get-pip.py

建议全都pip3安装
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






install miniconda for armv7:
https://stackoverflow.com/questions/39371772/how-to-install-anaconda-on-raspberry-pi-3-model-b

python -m pip install --upgrade pip
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

pip install --upgrade pip

conda create -name your_conda_env python=3.5
source activate your_conda_env



# how to install rknntookitlite
https://dev.t-firefly.com/thread-104204-1-1.html
# how to pack rootfs
https://dev.t-firefly.com/thread-118433-1-3.html
https://wiki.t-firefly.com/zh_CN/Firefly-RK3399/export_dev_sf.html