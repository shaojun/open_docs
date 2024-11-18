please help to create a web service by using these framework:
* python3.11
* python async io
* python fastapi
* python logging module
  with split log file by date and size, the default split size is 10MB.

should implement these APIs:
## dtu services
### get_dtu_state
  parameters is the string type `dtu_sn`.
  the return value is DTU_DEVICE_STATE enum type with values: `Online`, `Offline`, `Unknown`.    

## sub device services
### get_sub_device_state
  parameters is the string type `device_id`.
  the return value is SUB_DEVICE_STATE enum type with values: `Online`, `Offline`, `Unknown`, `Unsupport`.
### send_sub_device_request
  parameters is a class of `SubDeviceRequest` type with structure: `{"id":"sdfsdfdsfsd", "sub_device_id":"2", "type":"GPS_query", "data":{}, "description":"for read data"}`, or `{"id":"sdfsdfdsfsd", "sub_device_id":"1", "type":"Probe_reading", "data":{}, "description":"for read data"}`
  the return value is a class of `SubDeivceResponse` type with structure: `{"id":"3wewew","sub_device_id":"2", "type":"GPS_query", "data":{} }`
