# filebeat for uploading device hub statistics log to aliyun log service    
create the filebeat config file:
```
sudo nano /etc/filebeat/devicehub_eb_watch.yml
```
input below content:
```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /home/shao/tao-toolkit-triton-apps/tao_triton/python/device_hub/log/global_statistics.log
    # does not work??- /home/shao/tao-toolkit-triton-apps/tao_triton/python/device_hub/log/*/electric>processors:
  - dissect:
      tokenizer: '%{service.event_date} %{service.event_time}-[%{service.loglevel}]%{service.board_id>      field: "message"
      target_prefix: "dissect"

  #- decode_json_fields:
  #    fields: ["dissect.json_message"]
  #    target: ""

  - drop_fields:
      fields: ["message"]
      ignore_missing: true
output.kafka:
  enabled: true
  hosts: ["devicehub-eb-watch.cn-shanghai.log.aliyuncs.com:10012"]
  username: "devicehub-eb-watch"
  password: "LTAI5tEC6fLcrKSs7wwZdJ2X#QDaUPycyawar7t3b2qyeImdKz4X7A6"
  ssl.certificate_authorities:
  topic: 'logstorefordevicehubebwatch.json'
  headers:
    - key: "data-parse-format"
      value: "json"
  partition.hash:
    reachable_only: false
```
run the filebeat with the config file:
```
sudo filebeat run -c /etc/filebeat/devicehub_eb_watch.yml -e
```
