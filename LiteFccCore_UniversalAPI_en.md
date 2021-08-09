# LiteFccCore Overview


LiteFccCore is a software application (also called as **FCC** in fueling industry), it runs on a PC with I/O Ports(like serial port, Ethernet), the FCC internally integrated(implemented) many device communication protocols, by using I/O Ports, it can communicating with different devices(Dispenser, ATGs, Sensors) with protocol defined messages.

LiteFccCore is designed for modularization, different functions are implemented as PlugIn(s) internally, for most sceniaro, the developer or the user are work or use for a PlugIn, each PlugIn can providing a configuration user interface(Config UI), and some other functions.



There're 2 types of plugin:

 - Device Drivers

Each driver is responsible for communicating with a specific external device, it does not contain business logic(or very limited), just simply using the device protocol msg to communicate with device, and providing basic functions to higher level Apps.

Take `WayneDartPump` driver for example, it implemented `Wayne Dart` protocol, the protocol defined basic function for a dispenser, like read pump status, authorize pump, change pump price and etc., the `driver` just open these function interface, and wait for external to call.

 - Apps

Each App is for a specific business scenario, and mostly binding to a specific business project or product, it's sitting above the `Device Driver`, thus can use the `Device Driver` to compose any business logic and form a complete product features.

Take `IFSF-FDC-POS Server` app for example, it opens a TCP port to wait external POS to connect in, also, it defined several standard msg according from `IFSF Forum`, implemented `Wayne Dart` protocol, the protocol defined basic function for a dispenser, like read pump status, authorize pump, change pump price and etc., the `driver` just open these function interface, and wait for external to call.


 ### Overall Arch


![Image description](https://images.gitee.com/uploads/images/2021/0809/100650_96fac195_8024409.png "屏幕截图.png")

can see the `Device Drviers` are the most underlying parts, the higher level `Apps` can use `Drivers` to compose specific business logic, and both of them can implement the `Universal API` feature for expose functions to external, there're 2 major internal features are using the `Universal API`, they're:

 - Config UI

Providing user interface to allow user input parameters for a PlugIn to initialize:

![Image description](https://images.gitee.com/uploads/images/2021/0809/100530_14e45f41_8024409.png "屏幕截图.png")

 - Web Console

Providing user interface to show the status of the internal functions(like a control panel):

![Image description](https://images.gitee.com/uploads/images/2021/0809/101852_2e3179e9_8024409.png "屏幕截图.png")


The `Universal API` can be called directly by external any applications, like the POS.




# Universal API

The `Device Driver` or the `Apps` can expose their function to external, the external can access these function by APIs, which is call Universal API.

> API support to be called via: **HTTP(webapi), WebSocket, MQTT**


## Discover API

Depends on the release of the LiteFccCore or the configuration, each instance of LiteFccCore may providing different API set, even for a same API, it may get changed by different release of LiteFccCore, so the external caller side should treat the APIs are a `dynamic resource`, thus every time it get disconnected with LiteFccCore, it should re-run the discover procedure to get the correct API list and or other call details.

LiteFccCore provided one only fixed API which is the **Discover API** (also called `show me api`), external should always access this API first to get other API list.

You can access the discover API by a browser[human readable version](http://localhost:8384/Home/ShowMeApi?apiType=localmqtt&tags=,&format=pretty)

or another version that used for application:

* use HTTP POST to discover the API

below sample is to get all available APIs with `webapi` format:
```
URL: http://localhost:8384/u/?apitype=service&an=ShowMeApi&pn=ProcessorsDispatcher&en=Edge.Core.Processor.Dispatcher.DefaultDispatcher
Content-Type: application/json
Content: ["webapi",[]]"  
```

below sample is to get all available APIs with `mqtt` format:
```
URL: http://localhost:8384/u/?apitype=service&an=ShowMeApi&pn=ProcessorsDispatcher&en=Edge.Core.Processor.Dispatcher.DefaultDispatcher
Content-Type: application/json
Content: ["localmqtt",[]]
```

if you want to limit the API by targeting a specific Device or tag, then you need pass in `API TAG`, below sample is to only get <i>Pump</i>(dispenser) releated APIs:
```
URL: http://localhost:8384/u/?apitype=service&an=ShowMeApi&pn=ProcessorsDispatcher&en=Edge.Core.Processor.Dispatcher.DefaultDispatcher
Content-Type: application/json
Content: ["localmqtt",["Pump"]] 
```

* use MQTT to discover the API

mqtt server url:  mqtt://127.0.0.1:8388
publish to(replace the `+` to a local unique id to simulate RPC call):
```
Topic: /sys/Edge.Core.Processor.Dispatcher.DefaultDispatcher/ProcessorsDispatcher/thing/service/ShowMeApi/+
Content: ["localmqtt",[]]
```
subscribe to(replace the `+` to above value used in publish):
```
Topic: /sys/Edge.Core.Processor.Dispatcher.DefaultDispatcher/ProcessorsDispatcher/thing/service/ShowMeApi_reply/+
```


below is a sample that using a 3rd party mqtt client tool to call the `show me api`:


![Image description](https://images.gitee.com/uploads/images/2021/0705/110753_6a2d740b_8024409.png "屏幕截图.png")

## Resolve the result from Show Me API

the result of the Discover API is a below data structure, each element stands for an API.
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
`providerType` is the PlugIn name of the API.

`apiName` is the name of API, apiName in each providerType is unique.

`baseCategory` is a category of an API, there're 3 types of category, `service` is for calling from external, like a rpc function, `event` is triggered from FCC, external just wait for the notification, like a device entered an invalid state, then a event will be fired and notify external.

`path` is the full address of an API,if baseCategory is `service`, and external is using mqtt,then publish to this path to start a service call. if it's a `event`, then just  subscribe this topic to wait for notification.

`inputParametersJsonSchemaStrings` is the json schema for the input paramters,when call the API, the input parameter must satisfy the schema,if the schemal is null, then means this API does not support input parameter.

`inputParametersExampleJson` a sample for input parameter, help external to understand how to call the API.

`outputParametersJsonSchema` is the json schema for the output result, the external can know what the result structure would be.



## USE API

* Contact with developer to know what is the `tag`,`providerType`,`apiName` of the targeting API
* Call `showmeapi` API to discover based on step1's tag name, read schema info,and prepare the input parameter.
* Start the call.