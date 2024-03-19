# 现状 
现有边缘的远程连接方案是**frp预配置静态端口** + **长连接**的经典`frp`方式.
边缘与云端是没有控制链路的.

# 问题

## 无法支撑更多数量的连接
现有方案会使用服务器端的大量端口,并且由于`frp server`与`frp client`是处于长连接状态,也无法支持client的大数量增长需求.

> 到24年3月份, 边缘数量400台左右, 已经定时开始出现frp server无法接受客户端请求的错误的情况了

## 远程用户体验较差
另外根据自身体验和客户侧反馈,现有的对边缘进行远程连接以进行维护工作的步骤也比较麻烦, 即当维护人员要进边缘`shell`时,得先到`web运维端`上查看预分配给目标边缘的`ssh端口`, 再手动运行`putty`再进行`ssh`连接.

## 支持未来新的业务场景
边缘将会有更多的场景需要与云端通讯, 包括:
* 从云端远程配置边缘
* 从云端控制边缘
* 从云端查询边缘信息

# 技术方案
## 基础
通过kafka来实现云端与边缘的通讯, 需要定义到**单个边缘粒度**的`kafka topic`, 并对其中的消息进行分类:
* Request    
  从云端向边缘, 或者相反, 它表示需要对方完成一个动作并回复.
  Request应该要分几个大的类型, 以便复用于各种应用场景中.
  Request应有id,并在Response中也携带.
  主体结构如下:
  ```
  {"request":{"id":111,"sender":"web_backend","timestamp":"","description":"setup and start frp", type="change_frp_client_config" "body":{}}}
  ```
而根据不同的应用和目的,应对`body`部分进行自定义.
* Response    
  对Request的回应.
  主体结构如下:
  ```
  {"response":{"id":111,"sender":"edge_board","timestamp":"","description":"response for setup and start frp","overall_status_code":"200","overall_status":"Succeed",  "body":{}}}
  ```
* Event    
  某方主动发送给对方的, 不需要对方回复的一类消息.
  ```
  {"event":{"id":111,"sender":"edge_board","timestamp":"","description":"I'm about to crash", "body":{}}}
  ```
## 应用开发
### frp新远程方案
#### 需求
* frp client的配置将是动态下载的    
即**web后端**会将**frp client所需要的配置**内容通过`Request`发送, client获取kafka消息后将保存此配置信息.
* frp client的启停将是动态的    
即**web后端**会发送启动或者停止**frp client**的命令, 仍然是通过`Request`发送, 而边缘上的frp client默认处理关闭状态, 在获取到启动或者停止命令后, 再启动或者停止frp client服务.
* 支持web界面上的开启或者停止指定边缘上的frp client操作
* 支持web界面上的web ssh操作

综合上方, 新的界面请参考:    
![image](https://github.com/shaojun/open_docs/assets/3241829/3cbb11a7-688f-4976-9e66-f4d7de2f10e6)

#### 技术实现
在web界面上点击后,**web后端**会发送一个向指定topic的kafka消息，其中有一系列（用于摄像头，边缘ssh，fcc web config)的动态frp端口.
边缘上的watch dog收到后，先判断此消息的时间戳，如果是很久以前的，比如十分钟前的，则直接丢弃。
否则生成frp client新配置，再启动frp client service, 同时回复kafka消息以告知云端状态。
如果边缘后续收到关闭frp client service的消息则马上关闭服务，同时回复云端，否则过n分钟后（由下载配置信息中指定，可以默认120分钟），自动关闭。

以下为Request中的body示例:
```
"body":[{"edge_ssh_remote_port":3997,"edge_fcc_web_config_page_remote_port":3998 "camera_admin_web_page_remote_port":3999,
"auto_stop_service_in":3600}],
"actions":[{"name":"start_frp_client_service"}]}
```

### 获取边缘CPU使用量
...
...

### 请求边缘上传视频片段
...
...
