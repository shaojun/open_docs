### kafka dependency on Jetson
We are going to run deepstream-test4 in this example. First install `librdkafka`


```
cd /opt/nvidia/deepstream/deepstream/sources/libs/kafka_protocol_adaptor
sudo git clone https://github.com/edenhill/librdkafka.git
cd librdkafka
sudo git reset --hard 7101c2310341ab3f4675fc565f64f0967e135a6a
sudo ./configure
sudo make
sudo make install
sudo cp /usr/local/lib/librdkafka* /opt/nvidia/deepstream/deepstream/lib/
sudo ldconfig
#sudo apt-get install libglib2.0 libglib2.0-dev
#sudo apt-get install  libjansson4  libjansson-dev
```
Now librdkafka is installed, we will start out deepstream-application with proper configuration.

`cd /opt/nvidia/deepstream/deepstream/sources/apps/sample_apps/deepstream-test4`

before compile, need set env variable, For Jetson: `export CUDA_VER=10.2` , For x86: `export CUDA_VER=11.4`

start compile the app:

`sudo -E make`

after a while, you should see the file `./deepstream-test4-app` is created at the current folder.

now, run the test demo on the app just compiled, it just send detected data to kafka server on topic `test`:


 **make sure the video file is h.264 encoded, otherwise the app may hang!!**
 **_replace the `<ADD-YOUR-HOST-IP-HERE>` with your kafka server's ip or url domain name value_** 


```
./deepstream-test4-app -i /opt/nvidia/deepstream/deepstream/samples/streams/sample_720p.h264 \
    -p /opt/nvidia/deepstream/deepstream/lib/libnvds_kafka_proto.so \
    --conn-str="<ADD-YOUR-HOST-IP-HERE>;9092" \
    --topic="test" -s 0
```

Output:

```
Now playing: /opt/nvidia/deepstream/deepstream/samples/streams/sample_720p.h264

Using winsys: x11 
Opening in BLOCKING MODE 
ERROR: Deserialize engine failed because file path: /opt/nvidia/deepstream/deepstream-5.0/sources/apps/sample_apps/deepstream-test4/../../../../samples/models/Primary_Detector/resnet10.caffemodel_b1_gpu0_int8.engine open error
0:00:01.507939438 10139   0x55afb59a90 WARN                 nvinfer gstnvinfer.cpp:616:gst_nvinfer_logger:<primary-nvinference-engine> NvDsInferContext[UID 1]: Warning from NvDsInferContextImpl::deserializeEngineAndBackend() <nvdsinfer_context_impl.cpp:1690> [UID = 1]: deserialize engine from file :/opt/nvidia/deepstream/deepstream-5.0/sources/apps/sample_apps/deepstream-test4/../../../../samples/models/Primary_Detector/resnet10.caffemodel_b1_gpu0_int8.engine failed
0:00:01.508133807 10139   0x55afb59a90 WARN                 nvinfer gstnvinfer.cpp:616:gst_nvinfer_logger:<primary-nvinference-engine> NvDsInferContext[UID 1]: Warning from NvDsInferContextImpl::generateBackendContext() <nvdsinfer_context_impl.cpp:1797> [UID = 1]: deserialize backend context from engine from file :/opt/nvidia/deepstream/deepstream-5.0/sources/apps/sample_apps/deepstream-test4/../../../../samples/models/Primary_Detector/resnet10.caffemodel_b1_gpu0_int8.engine failed, try rebuild
0:00:01.508171504 10139   0x55afb59a90 INFO                 nvinfer gstnvinfer.cpp:619:gst_nvinfer_logger:<primary-nvinference-engine> NvDsInferContext[UID 1]: Info from NvDsInferContextImpl::buildModel() <nvdsinfer_context_impl.cpp:1715> [UID = 1]: Trying to create engine from model files
INFO: [TRT]: Reading Calibration Cache for calibrator: EntropyCalibration2
INFO: [TRT]: Generated calibration scales using calibration cache. Make sure that calibration cache has latest scales.
INFO: [TRT]: To regenerate calibration cache, please delete the existing one. TensorRT will generate a new calibration cache.
INFO: [TRT]: 
INFO: [TRT]: --------------- Layers running on DLA: 
INFO: [TRT]: 
INFO: [TRT]: --------------- Layers running on GPU: 
INFO: [TRT]: conv1 + activation_1/Relu, block_1a_conv_1 + activation_2/Relu, block_1a_conv_2, block_1a_conv_shortcut + add_1 + activation_3/Relu, block_2a_conv_1 + activation_4/Relu, block_2a_conv_2, block_2a_conv_shortcut + add_2 + activation_5/Relu, block_3a_conv_1 + activation_6/Relu, block_3a_conv_2, block_3a_conv_shortcut + add_3 + activation_7/Relu, block_4a_conv_1 + activation_8/Relu, block_4a_conv_2, block_4a_conv_shortcut + add_4 + activation_9/Relu, conv2d_cov, conv2d_cov/Sigmoid, conv2d_bbox, 
INFO: [TRT]: Detected 1 inputs and 2 output network tensors.
ERROR: Serialize engine failed because of file path: /opt/nvidia/deepstream/deepstream-5.0/samples/models/Primary_Detector/resnet10.caffemodel_b1_gpu0_int8.engine opened error
0:00:15.193611535 10139   0x55afb59a90 WARN                 nvinfer gstnvinfer.cpp:616:gst_nvinfer_logger:<primary-nvinference-engine> NvDsInferContext[UID 1]: Warning from NvDsInferContextImpl::buildModel() <nvdsinfer_context_impl.cpp:1743> [UID = 1]: failed to serialize cude engine to file: /opt/nvidia/deepstream/deepstream-5.0/samples/models/Primary_Detector/resnet10.caffemodel_b1_gpu0_int8.engine
INFO: [Implicit Engine Info]: layers num: 3
0   INPUT  kFLOAT input_1         3x368x640       
1   OUTPUT kFLOAT conv2d_bbox     16x23x40        
2   OUTPUT kFLOAT conv2d_cov/Sigmoid 4x23x40         

0:00:15.204773224 10139   0x55afb59a90 INFO                 nvinfer gstnvinfer_impl.cpp:313:notifyLoadModelStatus:<primary-nvinference-engine> [UID 1]: Load new model:dstest4_pgie_config.txt sucessfully
Running...
NvMMLiteOpen : Block : BlockType = 261 
NVMEDIA: Reading vendor.tegra.display-size : status: 6 
NvMMLiteBlockCreate : Block : BlockType = 261 
Frame Number = 0 Vehicle Count = 3 Person Count = 2
Frame Number = 1 Vehicle Count = 3 Person Count = 2
Frame Number = 2 Vehicle Count = 3 Person Count = 2
Frame Number = 3 Vehicle Count = 4 Person Count = 3
Frame Number = 4 Vehicle Count = 4 Person Count = 2
Frame Number = 5 Vehicle Count = 3 Person Count = 2
Frame Number = 6 Vehicle Count = 3 Person Count = 2
Frame Number = 7 Vehicle Count = 4 Person Count = 2
Frame Number = 8 Vehicle Count = 4 Person Count = 2
Frame Number = 9 Vehicle Count = 4 Person Count = 2
Frame Number = 10 Vehicle Count = 4 Person Count = 2
Frame Number = 11 Vehicle Count = 5 Person Count = 2
Frame Number = 12 Vehicle Count = 4 Person Count = 2
Frame Number = 13 Vehicle Count = 4 Person Count = 2
Frame Number = 14 Vehicle Count = 5 Person Count = 2
Frame Number = 15 Vehicle Count = 5 Person Count = 2
Frame Number = 16 Vehicle Count = 4 Person Count = 2
Frame Number = 17 Vehicle Count = 5 Person Count = 2
```

If everything works fine, the application will start. You can use consumer.py to receive messages.

`python3 consumer.py`

Output:

```
{
   "messageid":"bf45b525-1f51-4018-b559-5d9017c32d31",
   "mdsversion":"1.0",
   "@timestamp":"2020-12-09T14:32:56.178Z",
   "place":{
      "id":"1",
      "name":"XYZ",
      "type":"garage",
      "location":{
         "lat":30.32,
         "lon":-40.55,
         "alt":100.0
      },
      "aisle":{
         "id":"walsh",
         "name":"lane1",
         "level":"P2",
         "coordinate":{
            "x":1.0,
            "y":2.0,
            "z":3.0
         }
      }
   },
   "sensor":{
      "id":"CAMERA_ID",
      "type":"Camera",
      "description":"\"Entrance of Garage Right Lane\"",
      "location":{
         "lat":45.293701447,
         "lon":-75.8303914499,
         "alt":48.1557479338
      },
      "coordinate":{
         "x":5.2,
         "y":10.1,
         "z":11.2
      }
   },
   "analyticsModule":{
      "id":"XYZ",
      "description":"\"Vehicle Detection and License Plate Recognition\"",
      "source":"OpenALR",
      "version":"1.0",
      "confidence":-0.10000000149011612
   },
   "object":{
      "id":"-1",
      "speed":0.0,
      "direction":0.0,
      "orientation":0.0,
      "vehicle":{
         "type":"sedan",
         "make":"Bugatti",
         "model":"M",
         "color":"blue",
         "licenseState":"CA",
         "license":"XX1234",
         "confidence":-0.10000000149011612
      },
      "bbox":{
         "topleftx":1062,
         "toplefty":472,
         "bottomrightx":1107,
         "bottomrighty":498
      },
      "location":{
         "lat":0.0,
         "lon":0.0,
         "alt":0.0
      },
      "coordinate":{
         "x":0.0,
         "y":0.0,
         "z":0.0
      }
   },
   "event":{
      "id":"f6836b64-ba79-48fe-9487-6872b5e9dfd7",
      "type":"moving"
   },
   "videoPath":""
}
```

Notes:
- The listPlease read about kafka basics before trying to create your infrastructure, it will help a lot in debugging.
- The listUse different group ids if you are planning to subscribe to same data from different consumers.
- The listIn case you want to balance the message in between consumers such that no two consumers get the same message, use same group id for both. However you need to create partitions on the topic.



