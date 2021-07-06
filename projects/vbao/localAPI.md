### Descrption
v宝在站级的一后台服务器将在感应到车主的车载信标后,通过 **本地LAN** 发送一个通知到我方中控机程序(以下简称 **FCC** )中,FCC将负责后续的流程,包括短码生成,油机交易控制,并最后将交易完成的结果通过 **本地LAN**再次发回给V宝站级后台. 

## 流程图

### 会员注册流程

注册的流程始于司机申请v宝车载设备,司机信息由v宝后台收集后, **主动** 发往稳牌服务器端

![Image description](https://images.gitee.com/uploads/images/2021/0705/113608_50aa4a32_8024409.png "屏幕截图.png")


### 无感加油流程

![Image description](https://images.gitee.com/uploads/images/2021/0705/113630_b178d936_8024409.png "屏幕截图.png")

## FCC API定义

`api tag`: `V宝`

`ProviderType`: `VBaoProxyApp.App`

API描述的示例(以实际运行时为准):

![Image description](https://images.gitee.com/uploads/images/2021/0706/172348_81aba4a5_8024409.png "屏幕截图.png")

![Image description](https://images.gitee.com/uploads/images/2021/0706/170945_dfb7be75_8024409.png "屏幕截图.png")