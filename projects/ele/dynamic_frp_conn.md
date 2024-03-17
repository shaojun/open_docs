# Overview 
现有方案是**静态预配置端口** + **长连接**,会使用大量端口,并frp server与client是处于长连接状态,无法支持client的增长需求.
同时, 当维护人员要进边缘shell时,不得不先到web上查看预分配的端口, 再手动开启putty再进行ssh连接,操作麻烦.

同时, 边缘将会有更多的场景需要从**外部获取参数**,也需要有一条与后端的**配置通路**.

# 需求a
* frp client的配置将是动态下载的    
即**web后端**会将**frp client所需要的配置**内容通过一个kafka消息定义和发送, client获取kafka消息后将保存此配置信息.
* frp client的启停将是动态的
即**web后端**会发送启动或者停止**frp client**的命令, 仍然是通过一个kafka消息定义和发送, 而边缘上的frp client默认处理关闭状态, 在获取到启动或者停止命令后, 再启动或者停止frp client服务.
* 支持web界面上的开启或者停止指定边缘上的frp client操作
* 支持web界面上的web ssh操作

综合上方, 新的界面请参考:    
![image](https://github.com/shaojun/open_docs/assets/3241829/3cbb11a7-688f-4976-9e66-f4d7de2f10e6)

# 技术实现
在web界面上点击后,**web后端**会发送一个向指定topic的kafka消息，其中有一系列（用于摄像头，边缘ssh，fcc web config)的动态frp端口.
边缘上的watch dog收到后，先判断此消息的时间戳，如果是很久以前的，比如十分钟前的，则直接丢弃。
否则生成frp client新配置，再启动frp client service, 同时回复kafka消息以告知云端状态。
如果边缘后续收到关闭frp client service的消息则马上关闭服务，同时回复云端，否则过n分钟后（由下载配置信息中指定，可以默认120分钟），自动关闭。

> 应注意, 因为考虑到未来的通用参数下载,所以消息体请进行考虑和设计,而不仅仅是用于frp此场景.
> 例如:
> {"sender":"web_backend", "receiver":"edge_watchdog", "timestamp":"2024-4-1 9:01:01", "description":"setup and start frp",
> "parameters":[{"edge_ssh_remote_port":3998, "camera_admin_web_page_remote_port":3999}]}
