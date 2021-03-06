# GinAdmin
这个项目是以Gin框架为基础搭建的后台管理平台，虽然很多人都认为go是用来开发高性能服务端项目的，但是也难免有要做web管理端的需求，总不能再使用别的语言来开发吧。所以整合出了GinAdmin项目，请大家多提意见指正！
1
![logo](README/logo.jpg)

![logo](README/index.jpg)

## 依赖
* golang > 1.8
* Gin
* BootStrap
* LayUi
* WebUpload
* [Light Year Admin Using Iframe](#https://gitee.com/yinqi/Light-Year-Admin-Using-Iframe)

## 功能清单

:white_check_mark:权限控制

:white_check_mark:日志管理

:white_check_mark:模板页面

:white_check_mark:自动分页

:white_check_mark:Docker部署

:black_square_button:命令行操作​

## 使用文档
- [开始使用](#开始使用)
- [docker-compose构建环境](#docker-compose)
- [项目目录](#结构)
- [分页](#分页)
- [日志](#日志)
- [数据库](#数据库)
- [定时任务](#定时任务)
- [配置文件](#配置文件)
- [模板页面](#模板页面)
- [用户权限](#用户权限)
- [API文档](#API文档)
- [线上部署](#线上部署)

### :small_blue_diamond:<a name="开始使用">开始使用</a>

1. git 克隆地址 

   ```
   git clone https://github.com/gphper/ginadmin.git
   ```

2. 下载依赖包

   ```go
   go mod download
   ```

3. 配置 `configs/config.ini`文件

   ```
   [mysql]
   username=root
   password=123456
   database=db_beego
   host=127.0.0.1
   port=3306
   max_open_conn=50
   max_idle_conn=20
   [session]
   session_name=gosession_id
   [base]
   port=:8091
   ```

4. 运行 `go run main.go`访问地址 http://localhost:端口地址/admin/login。默认账户：admin  密码：111111

### :small_blue_diamond:<a name="docker-compose">docker-compose构建环境</a>

1. 替换conf目录下的配置项

   ```ini
   [mysql]
   username=docker
   password=123456
   database=docker_mysql
   host=localmysql
   port=3306
   max_open_conn=50
   max_idle_conn=20
   [session]
   session_name=gosession_id
   [base]
   host=0.0.0.0
   port=20010
   fill_data=true
   ```

2. 执行命令 docker-compose up

### :small_blue_diamond:<a name="结构">项目目录</a>

```
|--api  // Api接口控制器
|--build // 封装的公共方法
|--cmd  // 命令行工具
|--configs // 配置文件
|--deployments // docker-compose 部署文件
|--internal //核心代码
|--logs // 日志存放目录
|--pkg // 公共调用部分
|--web //视图静态文件
```

### :small_blue_diamond:<a name="分页">分页</a>

1.  使用 `pkg/comment/util.go` 里面的 `PageOperation` 进行分页
    ```go
    adminDb := models.Db.Table("admin_users").Select("nickname","username").Where("uid != ?", 1)
    adminUserData := comment.PageOperation(c, adminDb, 1, &adminUserList)
    ```
2.  在html中使用
    ```go
    {{ .adminUserData.PageHtml }}
    ```

### :small_blue_diamond:<a name="日志">日志</a>
1.  自定义日志 在 `pkg/loggers` 目录下新建logger
    ```
    参考 userlog.go 文件
    ```
2.  调用自定义的的logger写日志
    ```go
    loggers.UserLogger.Info("无法获取网址",
    zap.String("url", "http://www.baidu.com"),
    zap.Int("attempt", 3),
    zap.Duration("backoff", time.Second),)
    ```

### :small_blue_diamond:<a name="数据库">数据库</a>

1. models下定义的文件均需要实现 `TableName() string`  方法，并将实现该结构体的指针写入到 `GetModels` 方法中

   ```go
   func GetModels() []interface{} {
   	return []interface{}{
   		&AdminUsers{},
   		&AdminGroup{},
   	}
   }
   ```

2. model需要继承 BaseModle 并且实现 TableName 方法，如果需要初始化填充数据的话，需要实现 FillData() 方法，并将数据填充需要执行的代码写到函数体里。详情参照 AdminUsers

3. 数据库迁移,先使用`go install cmd\ginadmin-cli`安装ginadmin-cli 命令，执行命令行工具

   ```go
   ginadmin-cli db migrate
   ```

4. 数据填充，需在相应目录下实现 `FillData()` 方法执行如下命令

   ```go
   ginadmin-cli db seed
   ```

5. 可以通过设置 ini 配置文件中的 `fill_data`和`migrate_table` 分别控制程序重启时自动迁移数据表和填充数据

### :small_blue_diamond:<a name="定时任务">定时任务</a>

-    在 `pkg/cron/cron.go`  添加定时执行任务

### :small_blue_diamond:<a name="配置文件">配置文件</a>

1. 现在 `configs/config.go` 添加配置项的 struct 类型，例如

   ```go
   type AppConf struct {
   	BaseConf `ini:"base"`
   }
   type BaseConf struct {
   	Port string `ini:"port"`
   }
   ```

2. 在 `configs/config.ini` 添加配置信息

   ```
   [base]
   port=:8091
   ```

3. 在代码中调用配置文件的信息

   ```go
   configs.App.BaseConf.Port
   ```

### :small_blue_diamond:<a name="模板页面">模板页面</a>

- 所有的后台模板都写到 `web/views/template` 目录下面，并且分目录存储，调用时按照 `目录/模板名称` 的方式调用


### :small_blue_diamond:<a name="用户权限">用户权限</a>

- 菜单权限定义到 `internal/menu/menu.go` 文件下，定义完之后在用户组管理里面编辑权限

- casbin版集成了casbin权限管理框架，官方地址：[casbin](#https://casbin.org/docs/zh-CN/get-started)

- 框架中的常用方法定义在  `comment/auth/casbinauth/asbin.go` 文件中

- 在控制器中可用从 `gin.context` 获取登录用户信息

  ```go
  info,_ := c.Get("userInfo")
  ```

- template 中判断权限的函数 `judgeContainPriv` 定义在 `pkg/template/default.go` 文件下

  ```go
  "judgeContainPriv": func(username string, obj string, act string) bool {
  		if username == "admin" {
  			return true
  		}
  		ok, err := casbinauth.Check(username, obj, act)
  		if !ok || err != nil {
  			return false
  		}
  		return true
  },
  ```

### :small_blue_diamond:<a name="API文档">API文档</a>

- 使用 swagg 生成api文档，生成文件再docs目录下

  ```swag init -g cmd/ginadmin/main.go```
  
- 在根目录执行  `go build .\cmd\ginadmin\` 然后啊访问 http://localhost:20010/swagger/index.html

### :small_blue_diamond:<a name="线上部署">线上部署</a>

- 使用 `go build -tags=release .\cmd\ginadmin`  生成二进制文件

