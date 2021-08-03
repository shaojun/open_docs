# 关于通过 Universal API 来获取加油机数据与状态的流程
## 使用MQTT进行API服务发现

Mqtt Server Url:  mqtt://127.0.0.1:8388

> Service: ShowMeApi

publish to(请将+号换成唯一id以模拟RPC调用):
```
Topic: /sys/Edge.Core.Processor.Dispatcher.DefaultDispatcher/ProcessorsDispatcher/thing/service/ShowMeApi/+
Content: ["localmqtt",["IfsfFdcServer"]]
```
subscribe to(请将+号换成上一步publish时用的唯一id以模拟RPC调用):
```
Topic: /sys/Edge.Core.Processor.Dispatcher.DefaultDispatcher/ProcessorsDispatcher/thing/service/ShowMeApi_reply/+
```
## 使用API

 **API tag name: IfsfFdcServer** 

> 所有API的列表,以及输入,输出参数的解释,均可在以下动态文档中获取可读内容:
> http://localhost:8384/Home/ShowMeApi?apiType=localmqtt&tags=IfsfFdcServer,&format=pretty

 - 获取全站油枪布局信息

> Service: GetPumpsLayout

关于传入参数, 请放`[null]`即可,返回的sample如下:


```

[
  {
    "Name": "Bogus_Pump handler",
    "PumpId": 1,
    "Nozzles": [
      {
        "LogicalId": 1,
        "RealPriceOnPhysicalPump": 831,
        "SiteLevelNozzleId": 1,
        "ProductBarcode": 0,
        "ProductName": "0#"
      },
      {
        "LogicalId": 2,
        "RealPriceOnPhysicalPump": 831,
        "SiteLevelNozzleId": 2,
        "ProductBarcode": 2,
        "ProductName": "92#"
      }
    ],
    "AmountDecimalDigits": 2,
    "VolumeDecimalDigits": 2,
    "PriceDecimalDigits": 2,
    "VolumeTotalizerDecimalDigits": 2
  },
  {
    "Name": "Bogus_Pump handler",
    "PumpId": 2,
    "Nozzles": [
      {
        "LogicalId": 1,
        "RealPriceOnPhysicalPump": 831,
        "SiteLevelNozzleId": 3,
        "ProductBarcode": 0,
        "ProductName": "0#"
      },
      {
        "LogicalId": 2,
        "RealPriceOnPhysicalPump": 831,
        "SiteLevelNozzleId": 4,
        "ProductBarcode": 2,
        "ProductName": "92#"
      }
    ],
    "AmountDecimalDigits": 2,
    "VolumeDecimalDigits": 2,
    "PriceDecimalDigits": 2,
    "VolumeTotalizerDecimalDigits": 2
  }
]
```
示例json表示站点中有两个 Pump 即加油点,它们的 `PumpId`分别为1 and 2,且每个 Pump 下均有2个 Nozzle 即油枪.

`RealPriceOnPhysicalPump` 是指油枪上的真实价格, 可能为null.

`SiteLevelNozzleId` 是指油枪的站级枪号,全站唯一.

`ProductBarcode` 是指油枪上的油品的油品号码,应注意的是,每个站对于相同的油品名可能有着不同的油品号码.

`ProductName` 是指油枪上的油品的油品名称.

`AmountDecimalDigits` 油机的金额相关的读数的小数点位数, 从油机端读来的金额相关的(如交易总金额)是不带小数的,将原始值再对其进行小数点计算后,才是具有可读性的金额.

`VolumeDecimalDigits` 油机的升数相关的读数的小数点位数, 同上.

`PriceDecimalDigits` 油机的单价的小数点位数, 同上.

`VolumeTotalizerDecimalDigits` 油机的升数累积读数的小数点位数, 一般等同于`VolumeDecimalDigits`.

> 大多数情况下, POS 的油枪图标应根据上述示例中的 Nozzles 进行绘制.

 - 改油机价格

> Service: ChangeFuelPriceAsync

传入和传出参数请参考localmqtt pretty doc.

 - 收取油机状态改变通知

当油机处于提枪,放回枪,加油中等状态时,FCC将发送主动事件通知,以实时通知外界(如POS机端).

> Event: OnFdcControllerStateChange

传入和传出参数请参考localmqtt pretty doc.

`FDC_READY` 状态是指油枪未提,即空闲状态.

`FDC_CALLING` 状态是指油枪已经提起,即等待授权即可开始加油的状态.

`FDC_AUTHORISED` 状态是指油枪已经得到授权,马上即可开始加油的状态.

`FDC_FUELLING` 状态是指油枪正在加油中.

`FDC_OFFLINE` or `FDC_CLOSED` 状态是指油机与FCC的通讯中断,或者油机位于不可知的错误状态.

- 收取油机加油过程中的金额和升数数字变化的通知

> Event: OnCurrentFuellingStatusChange

当油机处于加油过程中的时候,加油机将持续将持续变化上升的金额和升数数值不断的传回,可通过此事件来实现获取这类数据.

- 收取油机交易状态改变通知

当油机的某笔交易的状态产生变化时,将主动发起此事件通知,以下为可能引起交易状态改变的原因:

    * 油机新加完一笔油后,将产生一笔待付交易,此交易的状态为`Payable`,所有POS机都可以对其进行锁定操作.

    * 某笔交易被某个POS锁定时, 此交易的状态变为`Locked`,其它POS机不可再对其进行锁定和清除.

    * 某笔交易被某个POS清除(一般代表成功支付了)了后,此交易的状态变为`Cleared`.

> Event: OnFdcFuelSaleTransactinStateChange

传入和传出参数请参考localmqtt pretty doc.

 - 锁定交易

> Service: LockFuelSaleTrxAndNotifyAllFdcClientsAsync

 - 清除交易

> Service: ClearFuelSaleTrxAndNotifyAllFdcClientsAsync

 - 主动查询待付交易

> Service: GetAvailableFuelSaleTrxsWithDetailsAsync
