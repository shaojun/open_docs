# filebeat for uploading device hub statistics log to aliyun log service    
https://help.aliyun.com/zh/sls/user-guide/use-the-kafka-protocol-to-upload-logs?spm=5176.2020520112.0.0.55413efd5MWjXH    
create the filebeat config file:
```
sudo nano /etc/filebeat/filebeat.yml
```
input below content:
```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /home/shao/tao-toolkit-triton-apps/tao_triton/python/device_hub/log/*/statistics.log
    # does not work??- /home/shao/tao-toolkit-triton-apps/tao_triton/python/device_hub/log/*/electricBicycleEnteringEventDetector.log
processors:
  - add_fields:
      fields:
        name: 'iam created by shao'
        id: '119119'
  - dissect:
      tokenizer: '%{service.event_date} %{service.event_time}-[%{service.loglevel}]%{service.board_id} | %{service.event_type} | %{service.event_data}'
      field: "message"
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
**direct run** the filebeat with the config file:
```
sudo filebeat run -c /etc/filebeat/filebeat.yml -e
```
or **run as service**:
```
sudo service filebeat start
```
