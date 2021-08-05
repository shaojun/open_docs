**LiteFccCore**

LiteFccCore is a software application which is also called as **FCC** in fueling industry, it runs on a PC with I/O Ports(like serial port, Ethernet), by using these Ports, the FCC can communicating with different devices(Dispenser, ATGs, Sensors).

LiteFccCore is internally implemented as many PlugIn(s), each plugin is take a charge to communicate with a specific external devices, thus, the FCC can read devices state, and operate devices.


**Universal API**

LiteFccCore 是作为平台目的设计的,它将对接的硬件功能进行简化和抽象,并制定了对外开放接口,外部应用程序可以访问这些接口,以快捷实现的对硬件的控制,并形成业务功能.

这类开放的接口称为 Universal API, 以下简称 API.

> API 支持外部通过**HTTP(webapi),WebSocket,MQTT**三种方式调用.

---

## 发现API

依 LiteFccCore 的具体配置(如连接了什么设备,开启了哪些功能),它所提供的**API集**将是不同的;同时,哪怕同一个 API,也因技术原因,在 LiteFccCore 每次重启后,它的调用路径也可能发生改变.

所以,外部应用程序应将API作为一种**动态资源**来对待,即在每次与 LiteFccCore 重新建立连接后,都应该重新进行API发现,以获得有效的 API 调用入口和方式列表.

为了给外部提供一个获取当前所支持的API列表功能, LiteFccCore 提供了一个固定的 API,即 **发现API**,外部应用程序应通过调用此 API,来发现其它**动态API**.

发现API 可以由通过浏览器访问一个[可读性的版本](http://localhost:8384/Home/ShowMeApi?apiType=localmqtt&tags=,&format=pretty)

同时也提供一个供程序获取的版本,以下做介绍:

* 使用HTTP POST进行API服务发现

以下示例将返回 webapi 格式的所有API信息
```
URL: http://localhost:8384/u/?apitype=service&an=ShowMeApi&pn=ProcessorsDispatcher&en=Edge.Core.Processor.Dispatcher.DefaultDispatcher
Content-Type: application/json
Content: ["webapi",[]]"  
```

以下示例将返回 mqtt 格式的 API 信息
```
URL: http://localhost:8384/u/?apitype=service&an=ShowMeApi&pn=ProcessorsDispatcher&en=Edge.Core.Processor.Dispatcher.DefaultDispatcher
Content-Type: application/json
Content: ["localmqtt",[]]
```

如果要指定设备类型,即只需要发现某些类别的API,则需要传入特定的 API TAG, 以下示例将返回<i>Pump</i>(油机)相关的API:
```
URL: http://localhost:8384/u/?apitype=service&an=ShowMeApi&pn=ProcessorsDispatcher&en=Edge.Core.Processor.Dispatcher.DefaultDispatcher
Content-Type: application/json
Content: ["localmqtt",["Pump"]] 
```

* 使用MQTT进行API服务发现

mqtt server url:  mqtt://127.0.0.1:8388
publish to(请将+号换成唯一id以模拟RPC调用):
```
Topic: /sys/Edge.Core.Processor.Dispatcher.DefaultDispatcher/ProcessorsDispatcher/thing/service/ShowMeApi/+
Content: ["localmqtt",[]]
```
subscribe to(请将+号换成上一步publish时用的唯一id以模拟RPC调用):
```
Topic: /sys/Edge.Core.Processor.Dispatcher.DefaultDispatcher/ProcessorsDispatcher/thing/service/ShowMeApi_reply/+
```


下图是用某mqtt client工具进行调用show me api的示例:


![Image description](https://images.gitee.com/uploads/images/2021/0705/110753_6a2d740b_8024409.png "屏幕截图.png")

## 解析API
发现 API 的返回结果是一个这样的数组结构, 每一个元素代表了一个 API.
```
[
{
	"providerType": "Applications.FDC.FdcServerHostApp",
	"providerTags": [
		"Pump",
		"IfsfFdcServer"
	],
	"providerConfigName": "Applications.FDC.FdcServerHostApp,FdcSer_ac8a48af9b2ce04d_0",
	"baseCategory": "Service",
	"apiName": "ChangeFuelPriceAsync_Reply",
	"path": "/Applications.FDC.FdcServerHostApp/Applications.FDC.FdcServerHostApp,FdcSer_ac8a48af9b2ce04d_0/thing/service/ChangeFuelPriceAsync_reply/+",
	"inputParametersExampleJson": "[1, 1]",
	"inputParametersJsonSchemaStrings": [
		"{\"title\": \"barcode\",\r\n  \"type\": \"integer\"\r\n}",
		"{\"title\": \"newPriceWithDecimalPoints\",\r\n  \"type\": \"number\"\r\n}"
	],
	"outputParametersExampleJson": null,
	"outputParametersJsonSchema": null,
	"description": ""
},
{
	"providerType": "Applications.FDC.FdcServerHostApp",
	"providerTags": [
		"Pump",
		"IfsfFdcServer"
	],
	"providerConfigName": "Applications.FDC.FdcServerHostApp,FdcSer_ac8a48af9b2ce04d_0",
	"baseCategory": "Event",
	"apiName": "OnFdcControllerStateChange",
	"path": "/Applications.FDC.FdcServerHostApp/Applications.FDC.FdcServerHostApp,FdcSer_ac8a48af9b2ce04d_0/thing/event/OnFdcControllerStateChange/post",
	"inputParametersExampleJson": null,
	"inputParametersJsonSchemaStrings": null,
	"outputParametersExampleJson": "{\r\n  \"newPumpState\": \"FDC_CONFIGURE\",\r\n  \"stateChangedNozzles\": [\r\n    {\r\n      \"pumpId\": 1,\r\n      \"physicalId\": 1,\r\n      \"logicalId\": 1,\r\n      \"realPriceOnPhysicalPump\": null,\r\n      \"expectingPriceOnFcSide\": null,\r\n      \"volumeTotalizer\": null,\r\n      \"logicalState\": \"FDC_CONFIGURE\"\r\n    }\r\n  ]\r\n}",
	"outputParametersJsonSchema": "{\r\n  \"definitions\": {\r\n    \"LogicalNozzle\": {\r\n      \"type\": [\r\n        \"object\",\r\n        \"null\"\r\n      ],\r\n      \"properties\": {\r\n        \"PumpId\": {\r\n          \"type\": \"integer\"\r\n        },\r\n        \"PhysicalId\": {\r\n          \"type\": \"integer\"\r\n        },\r\n        \"LogicalId\": {\r\n          \"type\": \"integer\"\r\n        },\r\n        \"RealPriceOnPhysicalPump\": {\r\n          \"type\": [\r\n            \"integer\",\r\n            \"null\"\r\n          ]\r\n        },\r\n        \"ExpectingPriceOnFcSide\": {\r\n          \"type\": [\r\n            \"integer\",\r\n            \"null\"\r\n          ]\r\n        },\r\n        \"VolumeTotalizer\": {\r\n          \"type\": [\r\n            \"integer\",\r\n            \"null\"\r\n          ]\r\n        },\r\n        \"LogicalState\": {\r\n          \"type\": [\r\n            \"string\",\r\n            \"null\"\r\n          ],\r\n          \"enum\": [\r\n            null,\r\n            \"FDC_CONFIGURE\",\r\n            \"FDC_DISABLED\",\r\n            \"FDC_ERRORSTATE\",\r\n            \"FDC_FUELLING\",\r\n            \"FDC_INVALIDSTATE\",\r\n            \"FDC_LOCKED\",\r\n            \"FDC_OFFLINE\",\r\n            \"FDC_OUTOFORDER\",\r\n            \"FDC_READY\",\r\n            \"FDC_REQUESTED\",\r\n            \"FDC_STARTED\",\r\n            \"FDC_SUSPENDED\",\r\n            \"FDC_CALLING\",\r\n            \"FDC_TEST\",\r\n            \"FDC_SUSPENDED_STARTED\",\r\n            \"FDC_SUSPENDED_FUELLING\",\r\n            \"FDC_CLOSED\",\r\n            \"FDC_AUTHORISED\",\r\n            \"FDC_UNDEFINED\"\r\n          ]\r\n        }\r\n      },\r\n      \"required\": [\r\n        \"PumpId\",\r\n        \"PhysicalId\",\r\n        \"LogicalId\",\r\n        \"RealPriceOnPhysicalPump\",\r\n        \"ExpectingPriceOnFcSide\",\r\n        \"VolumeTotalizer\",\r\n        \"LogicalState\"\r\n      ]\r\n    }\r\n  },\r\n  \"type\": \"object\",\r\n  \"properties\": {\r\n    \"NewPumpState\": {\r\n      \"type\": \"string\",\r\n      \"enum\": [\r\n        \"FDC_CONFIGURE\",\r\n        \"FDC_DISABLED\",\r\n        \"FDC_ERRORSTATE\",\r\n        \"FDC_FUELLING\",\r\n        \"FDC_INVALIDSTATE\",\r\n        \"FDC_LOCKED\",\r\n        \"FDC_OFFLINE\",\r\n        \"FDC_OUTOFORDER\",\r\n        \"FDC_READY\",\r\n        \"FDC_REQUESTED\",\r\n        \"FDC_STARTED\",\r\n        \"FDC_SUSPENDED\",\r\n        \"FDC_CALLING\",\r\n        \"FDC_TEST\",\r\n        \"FDC_SUSPENDED_STARTED\",\r\n        \"FDC_SUSPENDED_FUELLING\",\r\n        \"FDC_CLOSED\",\r\n        \"FDC_AUTHORISED\",\r\n        \"FDC_UNDEFINED\"\r\n      ]\r\n    },\r\n    \"StateChangedNozzles\": {\r\n      \"type\": [\r\n        \"array\",\r\n        \"null\"\r\n      ],\r\n      \"items\": {\r\n        \"$ref\": \"#/definitions/LogicalNozzle\"\r\n      }\r\n    }\r\n  },\r\n  \"required\": [\r\n    \"NewPumpState\",\r\n    \"StateChangedNozzles\"\r\n  ]\r\n}",
	"description": "When pump state changed, the event will fired"
}
]
```
`providerType`是提供 API 的插件名称,每个插件对接了不同的设备.

`apiName`是指 API 的名称,每个 providerType 中的 apiName 是唯一的,请用此值来唯一确定一个将要调用的 API.

`baseCategory`是指此 API 的分类,现有3种分类, `service`是指外部应用可以主动调用的,就像一个rpc函数, `event`是指由LiteFccCore主动发起的,外部则被动来接收,就是一个事件,如某硬件设备进入了异常状态,则会引发一个event.

`path`是指调用此 API 的路径,如果 baseCategory 是`service`,如外部应用使用 mqtt 形式的调用 ,则请直接调用此 path 进行 publish. 而对于 event,则 subscribe 它并等待通知.

`inputParametersJsonSchemaStrings`是指输入参数的 json schema,在调用时,输入的参数必须符合此规则,如果为 null,则说明此API不需要输入.

`inputParametersExampleJson`是指输入参数的示例 json 内容,以帮助调用者理解如何构建输入参数.

`outputParametersJsonSchema`是指输出结果的 json schema,外部应用可以根据它来预期返回值的结构.



## 使用API

* 与开发者联系,咨询所要访问的API的`tag`,`providerType`,`apiName`
* 通过 `showmeapi` API进行服务发现, 基于第一步中的信息,确定自己要调用的API,并通过发现结果以获取调用技术细节
* 发起实际调用