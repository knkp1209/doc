# Elasticsearch Kibana Filebeat



## 该文档主要说明 Filebeat 如何与 Elasticsearch Kibana 配合采集日志



## 1、基础环境搭建（可自行搭建，本文档基于 docker-compose 环境）

- ```shell
  mkdir /home/test
  cd /home/test
  # 基于 laradock 添加 filebeat 组件，也可以直接在本地安装 Filebeat 
  # laradock github 地址 https://github.com/laradock/laradock
  git clone https://github.com/knkp1209/laradock.git
  cd laradock 
  
  # 启动基础环境，首次启动会自动 build, 漫长的等待过程
  docker-compose up elasticsearch kibana filebeat
  ```



### 2、进入容器 Filebeat 配置 fields.yml

- ```shell
  cd /home/test/laradock
  
  docker-compose exec --user root filebeat /bin/bash
  
  vi ./fields.yml
  
  # 在字段 key.fields 下 增加如下配置 （yml 格式），见前后配置图对比
  ## vi 编辑
    - name: context.requestXml
      type: text
      description:
        type 为 text 才能模糊查找, 按自己的日志字段进行相关配置
  
    - name: context.raw_request
      type: text
      description:
        type 为 text 才能模糊查找, 按自己的日志字段进行相关配置
  
    - name: context.raw_response
      type: text
      description:
        type 为 text 才能模糊查找, 按自己的日志字段进行相关配置
  
  ## vi 编辑结束
  
  
  ```

- 配置前 fields.yml

![image-20211208192300565](https://knkp-doc.oss-cn-guangzhou.aliyuncs.com/doc/image-20211208192300565-20211208192444995.png) 

- 配置后 fields.yml

![image-20211208204931357](https://knkp-doc.oss-cn-guangzhou.aliyuncs.com/doc/image-20211208204931357.png)

### 3、进入容器 Filebeat 配置 filebeat.yml, 内容如下

- ```shell
  vi ./filebeat.yml
  ```

- ```shell
  #  filebeat.yml
  filebeat.config:
    modules:
      path: ${path.config}/modules.d/*.yml
      reload.enabled: false
  
  
  filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/*.log # 扫描日志路径
    json:
      keys_under_root: true
      overwrite_keys: true
      add_error_key: true
      message_key: nginx_request_id # 根据日志选择合适字段
  
  # 覆盖模板
  setup.template.overwrite: true
  
  processors:
    - add_cloud_metadata: ~
    - add_docker_metadata: ~
    - rename:
        fields:
         - from: "url" # url 字段被占用，这里进行重命名
           to: "_url"
    - timestamp: # 时间，用于 kibana 进行时间范围筛选
        field: datetime # 根据日志选择合适字段
        timezone: Asia/Shanghai
        layouts:
          - '2006-01-02 15:04:05.999'
        test:
          - '2019-06-22 16:33:51'
    - drop_fields:
        fields: ["log","host","input","agent","ecs","timestamp"]
        ignore_missing: false
  
  output.elasticsearch:
    hosts: '${ELASTICSEARCH_HOSTS:elasticsearch:9200}'
    username: '${ELASTICSEARCH_USERNAME:}'
    password: '${ELASTICSEARCH_PASSWORD:}'
  
  setup.kibana:
    host: 'kibana:5601'
    
  setup.template.overwrite: true
  setup.template.fields: "fields.yml"
  ```



### 4、生成 kibana 面板， 退出 Filebeat  容器，重启服务

- ```shell
  # 生成 kibana 面板
  ./filebeat setup
  exit
  docker-compose up elasticsearch kibana filebeat
  ```

5、将测试服务器日志复制到里面, 完成

- ```
  cp 文件路径/07-sf.fee.notify.log /home/test/logs/nginx

6、实际效果，公司环境可访问地址：http://172.16.50.83:5601/

![image-20211208200911803](https://knkp-doc.oss-cn-guangzhou.aliyuncs.com/doc/image-20211208200911803.png)



其他相关资料

https://www.elastic.co/guide/en/elasticsearch/reference/6.5/grok-processor.html

https://www.elastic.co/guide/en/beats/filebeat/current/elasticsearch-output.html

重新采集 

- 删除 index

  ```web-idl
  DELETE filebeat-7.9.1-2021.12.05-000001
  ```

  

- 删除 filebeat/data/registry

