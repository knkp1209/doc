# go-admin 开源项目 源码解读

### [项目地址](https://github.com/go-admin-team/go-admin)

### [官方文档介绍](https://doc.go-admin.dev):

- [go-admin](https://github.com/go-admin-team/go-admin) 是一个中后台应用框架，基于（gin, gorm, Casbin, Vue, Element UI）实现。go-admin 分为两个项目[go-admin](https://github.com/go-admin-team/go-admin) 和 [go-admin-ui](https://github.com/go-admin-team/go-admin-ui)

### 特性

- 开箱即用
- 遵循 RESTful API 设计规范
- 基于 GIN WEB API 框架，提供了丰富的中间件支持（用户认证、跨域、访问日志、追踪 ID 等）
- 基于 Casbin 的 RBAC 访问控制模型
- JWT 认证
- 支持 Swagger 文档(基于 swaggo)
- 基于 GORM 的数据库存储，可扩展多种类型数据库
- 配置文件简单的模型映射，快速能够得到想要的配置
- 代码生成工具
- 表单构建工具

### 内置功能

- 用户管理：用户是系统操作者，该功能主要完成系统用户配置。

- 部门管理：配置系统组织机构（公司、部门、小组），树结构展现支持数据权限。

- 岗位管理：配置系统用户所属担任职务。

- 菜单管理：配置系统菜单，操作权限，按钮权限标识等。

- 角色管理：角色菜单权限分配、设置角色按机构进行数据范围权限划分。

- 字典管理：对系统中经常使用的一些较为固定的数据进行维护。

- 参数管理：对系统动态配置常用参数。

- 操作日志：系统正常操作日志记录和查询；系统异常信息日志记录和查询。

- 登录日志：系统登录日志记录查询包含登录异常。

- 系统接口：根据业务代码自动生成相关的 api 接口文档。

- 代码生成：根据数据表结构生成对应的增删改查相对应业务，全部可视化编程。

- 表单构建：自定义页面样式，拖拉拽实现页面布局。

- 服务监控：查看一些服务器的基本信息。

### 上面是官方文档的一些介绍，具体可以去看官方文档，下面按自己的一些理解梳理一下这个项目的主要流程

- 首先使用 `go build`  构建了基于 [spf13/cobra](https://pkg.go.dev/github.com/spf13/cobra@v1.0.0) 库生成这个项目的一些命令（如：migrate, server 等）

- 然后运行 `./go-admin migrate -c=config/settings.dev.yml ` 迁移数据库

- 通过追踪上述的 `migrate` 命令，看下几个关键的点

  - 第一个 github.com/go-admin-team/go-admin-core/sdk/config.Config

    ```go
    // Config 配置集合
    type Config struct {
    	Application *Application          `yaml:"application"`
    	Ssl         *Ssl                  `yaml:"ssl"`
    	Logger      *Logger               `yaml:"logger"`
    	Jwt         *Jwt                  `yaml:"jwt"`
    	Database    *Database             `yaml:"database"`
    	Databases   *map[string]*Database `yaml:"databases"`
    	Gen         *Gen                  `yaml:"gen"`
    	Cache       *Cache                `yaml:"cache"`
    	Queue       *Queue                `yaml:"queue"`
    	Locker      *Locker               `yaml:"locker"`
    	Extend      interface{}           `yaml:"extend"`
    }
    ```

    

  - 第二个 github.com/go-admin-team/go-admin-core/sdk/config.Settings 这里上面的 Config 结构作为 Settings 结构里的一个属性

  - ```go
    type Settings struct {
    	Settings  Config `yaml:"settings"`
    	callbacks []func()
    }
    ```

  -  运行机制 

    ```go
    // 这里是很关键的点，是应用框架的起步, 调用了 config.Setup
    config.Setup(
    		file.NewSource(file.WithPath(configYml)), // 配置文件
    		initDB, // 这里是数据库的初始化
        ..., // 还可以一直增加其他的回调
    )
    
    // config 包的 Setup 函数 第一个参数为配置，第二个为变长参数，接收若干个回调函数
    // Setup 载入配置文件
    func Setup(s source.Source,
    	fs ...func()) {
    	_cfg = &Settings{
    		Settings: Config{
    			Application: ApplicationConfig,
    			Ssl:         SslConfig,
    			Logger:      LoggerConfig,
    			Jwt:         JwtConfig,
    			Database:    DatabaseConfig,
    			Databases:   &DatabasesConfig,
    			Gen:         GenConfig,
    			Cache:       CacheConfig,
    			Queue:       QueueConfig,
    			Locker:      LockerConfig,
    			Extend:      ExtendConfig,
    		},
    		callbacks: fs, // 回调方法
    	}
    	var err error
    	config.DefaultConfig, err = config.NewConfig(
    		config.WithSource(s),
    		config.WithEntity(_cfg),
    	)
    	if err != nil {
    		log.Fatal(fmt.Sprintf("New config object fail: %s", err.Error()))
    	}
      _cfg.Init() // 1. 调用外部可见初始化方法
    }
    
    func (e *Settings) runCallback() {
      // 依次执行 config.Setup 注入的回调方法
    	for i := range e.callbacks {
    		e.callbacks[i]() // 4. 控制反转
    	}
    }
    
    func (e *Settings) Init() {
    	e.init() // 2. 调用不可见的初始化方法
    	log.Println("!!! config init")
    }
    
    func (e *Settings) init() {
      // 初化始逻辑
    	e.Settings.Logger.Setup()
    	e.Settings.multiDatabase()
    	e.runCallback() // 3. 回调调度
    }
    
    // 初始化数据
    func initDB() {
      database.Setup()
    	fmt.Println("数据库迁移开始")
    	_ = migrateModel() // 代理了 gorm 的数据库迁移实现
    	fmt.Println(`数据库基础数据初始化成功`)
    }
    ```

    ### 总结：

    `./go-admin migrate -c=config/settings.dev.yml` 的执行流程：

    go-admin-core 将配置和要回调的函数注入到控制流中，控制流做完一些初始化的工作之后就将控制权反转给回调的函数列表，

    initDB 作为被回调的函数，它的作用是迁移数据库，用了 gorm 库的数据库迁移实现。

    