### GoLang 协程实现 MySQL 分段统计优化 (如按月统计等)

## 介绍：

- 业务场景：通过简单实现一个 MySQL 分段统计,  对比采用了 协程之后的总统计任务的耗时（协程也可以用于优化一些耗时的三方接口调用, 采用协程后遇到 IO 阻塞时就会挂起而不是原地等IO响应）

- 数据量：单表、数据记录数 

- 涉及知识点：协程与通道、GORM 库简单运用、数据库连接池

- ```sql
  # 表结构 这里主键ID 采用雪花算法生成，主要为了快速生成需要数据，自增ID会由于并发而导致ID重复，数据插入失败
  CREATE TABLE `user` (
    `id` bigint unsigned NOT NULL COMMENT 'ID, 雪花ID',
    `name` varchar(255) DEFAULT '' COMMENT '用户名',
    `age` tinyint DEFAULT '0' COMMENT '年龄',
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='用户表'
  ```

### Step1  数据库连接池实现（协程不能用同一个数据库连接, 所以要实现连接池）

```go
// 数据库连接池实现，这里没有采用 GORM 连接池，而是自己实现主要是业务比较简单，自己实现比较快；不要在生产环境使用

// 数据库连接数
const dsnTotal int64 = 40
var dsnList [dsnTotal]*gorm.DB

// 连接池惰性实现
func selectDsn(i int64) *gorm.DB {
	if dsnList[i] == nil {
		newLogger := logger.New(
			log.New(os.Stdout, "\r\n", log.LstdFlags), // io writer（日志输出的目标，前缀和日志包含的内容——译者注）
			logger.Config{
				SlowThreshold:             200 * time.Millisecond, // 慢 SQL 阈值
				LogLevel:                  logger.Silent,          // 日志级别
				IgnoreRecordNotFoundError: true,                   // 忽略ErrRecordNotFound（记录未找到）错误
				Colorful:                  false,                  // 禁用彩色打印
			},
		)
		// 参考 https://github.com/go-sql-driver/mysql#dsn-data-source-name 获取详情
		dsn := "root:12345678@tcp(127.0.0.1:3306)/test?charset=utf8mb4&parseTime=True&loc=Local"
		db, _ := gorm.Open(mysql.Open(dsn), &gorm.Config{
			Logger: newLogger,
			NamingStrategy: schema.NamingStrategy{
				SingularTable: true,
			},
		})
		dsnList[i] = db
	}
	return dsnList[i]
}
```



### Step2 利用通道实现等待机制（协程会随着程序的结束而消亡，这里需要 main() 中 等待协程运行结束）

```go
// 业务分段总数, 有多少个分段就会起多少个 goroutines
var total int64 = 35

// 每段数据量大小
var per int64 = 400000

// 定义一个带缓冲的通道，缓冲大小与业务分段总数一致
var ch = make(chan int, total)

func wait() {
	var i int64 = 0
	for _ = range ch {
		i++
		if i == total {
      // 协程都执行完成了，关闭通道
			close(ch)
		}
	}
}
```



### Step3 协程版分段查询实现

```go
func goroutine() {
	var i int64 = 0

	for i < total { // 分段总数
    // 起协程执行
		go func(i int64) {
      db := selectDsn(i % dsnTotal) // 从连接池中选择数据库连接
			var sumTotal string
			db.Raw("SELECT sum(id) as s from (SELECT id from `user` LIMIT ?, ?) as temp", i*per, (i+1)*per-1).Scan(&sumTotal)
			fmt.Println(i+1, sumTotal)
			ch <- 1
		}(i)
		//print(user.ID)
		i++
	}
}
```



### Step4 非协程版分段查询实现

```go
func normal() {
	var i int64 = 0

	for i < total {        // 分段总数
		db := selectDsn(i % dsnTotal) // 从连接池中选择数据库连接
		var sumTotal string
		db.Raw("SELECT sum(id) as s from (SELECT id from `user` LIMIT ?, ?) as temp", i*per, (i+1)*per-1).Scan(&sumTotal)
		fmt.Println(i+1, sumTotal)
		//print(user.ID)
		i++
	}
}
```



### Step5 执行对比

```go
func main() {
	fmt.Println("开始: ")
	t1 := time.Now()
	normal()
	elapsed := time.Since(t1)
	fmt.Println("耗时: ", elapsed)

	fmt.Println("开始协程: ")
	t1 = time.Now()
	goroutine()
	wait()
	elapsed = time.Since(t1)
	fmt.Println("耗时: ", elapsed)
}
```



### 对比结果，协程版耗时为 非协程的 50%，对一些 IO 等待时间长，但计算逻辑不复杂的业务，协程在耗时方面应该还会有更进一步的提升，如请求第三方接口等

<table style="margin-left: auto; margin-right: auto;">
    <tr>
        <td>
            <!--左侧内容-->
            <div>非协程, 执行时长  1m5.798042599s </div>
            <img src="https://knkp-doc.oss-cn-guangzhou.aliyuncs.com/doc/image-20220106190855427.png" />
        </td>
        <td>
            <!--右侧内容-->
            <div>协程版，执行时长 32.962620579s</div>
            <img src="https://knkp-doc.oss-cn-guangzhou.aliyuncs.com/doc/image-20220106191118698.png" />
        </td>
    </tr>
</table>

 [main.go](/Volumes/develop/learn/concurrent/main.go) 