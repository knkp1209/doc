# clickhouse 安装与基本使用

## 安装

### 安装服务端与客户端
```shell
# 创建数据存放文件夹
mkdir $HOME/some_clickhouse_database

# 启动 clickhouse 服务 （拉取服务端 docker 镜像并运行）
docker run -d --name some-clickhouse-server --ulimit nofile=262144:262144 -p 8123:8123 \
--volume=$HOME/some_clickhouse_database:/var/lib/clickhouse clickhouse/clickhouse-server

# 从客户端连接 （拉取客户端 docker 镜像并运行及配置关联服务端）
docker run -it --rm --link some-clickhouse-server:clickhouse-server clickhouse/clickhouse-client \
--host clickhouse-server
```

### 服务端与客户端安装完毕，（命令行客户端 及 web客户端 简单查询演示）

- ![image-20220212171211803](https://knkp-doc.oss-cn-guangzhou.aliyuncs.com/doc/image-20220212171211803.png)

- ![image-20220212172507379](https://knkp-doc.oss-cn-guangzhou.aliyuncs.com/doc/image-20220212172507379.png)



## 基本使用

### 数据导入

```shell
# 下载示例数据并解压到 clickhouse 的数据存放位置
cd $HOME
curl -O https://datasets.clickhouse.com/hits/partitions/hits_v1.tar
curl -O https://datasets.clickhouse.com/visits/partitions/visits_v1.tar
tar xvf hits_v1.tar -C ./some_clickhouse_database
tar xvf visits_v1.tar -C ./some_clickhouse_database

# 重启 clickhouse 服务, 使导入的数据生效
docker restart some-clickhouse-server
```

重启 clickhouse 服务后，查询可以看到已经成功导入相关数据了

![image-20220212173102190](https://knkp-doc.oss-cn-guangzhou.aliyuncs.com/doc/image-20220212173102190.png)

![image-20220212173225645](https://knkp-doc.oss-cn-guangzhou.aliyuncs.com/doc/image-20220212173225645.png)



### 创建数据库

```sql
# 查看已创建的所有数据库
SHOW DATABASES

# 如果数据库不存在则创建
CREATE DATABASE IF NOT EXISTS tutorial

# 对比结果
SHOW DATABASES
```



### 创建表，关键三要素 

1. 要创建的表的名称。
2. 表结构，例如：列名和对应的[数据类型](https://clickhouse.com/docs/zh/sql-reference/data-types/)。
3. [表引擎](https://clickhouse.com/docs/zh/engines/table-engines/)及其设置，这决定了对此表的查询操作是如何在物理层面执行的所有细节。

```sql
# 简单创建一个表
CREATE TABLE IF NOT EXISTS tutorial.example
(
		`EventDate` Date,
		`CounterID` UInt32,
    `UserID` UInt64,
    `Title` String
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID))
SAMPLE BY intHash32(UserID)
```



### 插入数据

```sql
# 简单插入一些数据，Date 类型插入时支持 时间戳 与 日期字段字符串
INSERT INTO tutorial.example (*) VALUES (1609430400, 100, 200, 'stringContent'), ('2022-01-01', 100, 200, 'stringContent');
```



查询数据

```sql
# 查询
select * from tutorial.example;
```

