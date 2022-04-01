# ELasticsearch Kibana FilebeatGET filebeat-7.9.1-2021.12.08-000001





```shell
# 
docker-compose exec --user root filebeat /bin/bash

# filebeat 终端
# filebeat.yml 新增 kibana 配置
vi filebeat.yml


```

```yaml
 - name: context.requestXml
    type: text
    description:
      Service that is the target of the event.

  - name: context.raw_request
    type: text
    description:
      Service that is the target of the event.

  - name: context.raw_response
    type: text
    description:
      Service that is the target of the event.

```



## 在 ELasticsearch Kibana Web 管理后台中的控制台添加管道 [点击进入后台](http://172.16.50.83:5601/app/dev_tools#/console)

### grok 处理 正则要转义

```json
PUT _ingest/pipeline/nginx-pipeline
{
  "description": "解析nginx日志",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": [
          "%{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration} %{TIMESTAMP_ISO8601:time}"
        ]
      }
    },
    {
      "date": {
        "field": "time",
        "target_field": "@timestamp",
        "formats": [
          "ISO8601"
        ],
        "timezone": "Asia/Shanghai"
      }
    }
  ]
}
```



```yml
output.elasticsearch:
 hosts: ["10.11.40.229:9200"]
 pipelines:
  \- pipeline: "mysql_error_pipeline"
   when.equals:
    fields.index: "mysql_error"
 indices:
  \- index: "mysql-error-%{+yyyy.MM.dd}"
   when.equals:
    fields.index: "mysql_error"

Docker IP: 172.17.0.2
```

```yml
filebeat.config:
  modules:
    path: ${path.config}/modules.d/*.yml
    reload.enabled: false


filebeat.inputs:
  enabled: true
  paths:
    - /usr/local/log/*.log

processors:
  - add_cloud_metadata: ~
  - add_docker_metadata: ~

output.elasticsearch:
  hosts: '${ELASTICSEARCH_HOSTS:elasticsearch:9200}'
  username: '${ELASTICSEARCH_USERNAME:}'
  password: '${ELASTICSEARCH_PASSWORD:}'
  pipeline: 'nginx-pipeline'

setup.kibana:
  host: 'kibana:5601'
```



```yml
PUT _ingest/pipeline/nginx-pipeline
{
  "description": "解析nginx日志",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": [
          "%{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration} %{TIMESTAMP_ISO8601:time}"
        ]
      }
    },
    {
      "date": {
        "field": "time",
        "target_field": "@timestamp",
        "formats": [
          "ISO8601"
        ],
        "timezone": "Asia/Shanghai"
      }
    }
  ]
}
```



### json

```yml
filebeat.config:
  modules:
    path: ${path.config}/modules.d/*.yml
    reload.enabled: false


filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /usr/local/log/*.log
  json:
    keys_under_root: true
    overwrite_keys: true
    add_error_key: true
    message_key: nginx_request_id

processors:
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  -  timestamp:
      field: timestamp
      timezone: Asia/Shanghai
      layouts:
        - '2006-01-02 15:04:05.999'
      test:
        - '2019-06-22 16:33:51.111'
  -  drop_fields:
      fields: ["log","host","input","agent","ecs","timestamp","url"]
      ignore_missing: false

output.elasticsearch:
  hosts: '${ELASTICSEARCH_HOSTS:elasticsearch:9200}'
  username: '${ELASTICSEARCH_USERNAME:}'
  password: '${ELASTICSEARCH_PASSWORD:}'

setup.kibana:
  host: 'kibana:5601'
```



其他相关资料

https://www.elastic.co/guide/en/elasticsearch/reference/6.5/grok-processor.html

https://www.elastic.co/guide/en/beats/filebeat/current/elasticsearch-output.html



### 重置采集

- filebeat/data/registry

filebeat.config:

 modules:

  path: ${path.config}/modules.d/*.yml

  reload.enabled: false





filebeat.inputs:

\- type: log

 enabled: true

 paths:

  \- /var/log/nginx/*.log

 json:

  keys_under_root: true

  overwrite_keys: true

  add_error_key: true

  message_key: nginx_request_id



processors:

 \- add_cloud_metadata: ~

 \- add_docker_metadata: ~

 \- rename:

   fields:

​    \- from: "url"

​     to: "_url"

 \- timestamp:

   field: datetime

   timezone: Asia/Shanghai

   layouts:

​    \- '2006-01-02 15:04:05.999'

   test:

​    \- '2019-06-22 16:33:51'

 \- drop_fields:

   fields: ["log","host","input","agent","ecs","timestamp"]

   ignore_missing: false



output.elasticsearch:

 hosts: '${ELASTICSEARCH_HOSTS:elasticsearch:9200}'

 username: '${ELASTICSEARCH_USERNAME:}'

 password: '${ELASTICSEARCH_PASSWORD:}'



setup.kibana:

 host: 'kibana:5601'
