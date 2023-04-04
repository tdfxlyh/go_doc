# 1.数据库相关操作

## (1)非orm框架



### ①引入

```shell
go get github.com/go-sql-driver/mysql
```



### ②初始化

```go
import (
	"database/sql"
	"fmt"

	"github.com/Project02/utils"
	_ "github.com/go-sql-driver/mysql"
)

func InitDB() (err error) {
	// DSN:Data Source Name
	dsn := "root:991113@tcp(127.0.0.1:3306)/chuangyitest?charset=utf8mb4&parseTime=True"
	// 不会校验账号密码是否正确
	// 注意！！！这里不要使用:=，我们是给全局变量赋值，然后在main函数中使用全局变量db
	MyDB, err = sql.Open("mysql", dsn)
	if err != nil {
		fmt.Println(fmt.Sprintf("database open err, err=%s", utils.GetLogStr(err)))
		return
	}
	// 尝试与数据库建立连接（校验dsn是否正确）
	err = MyDB.Ping()
	if err != nil {
		fmt.Println(fmt.Sprintf("database ping err, err=%s", utils.GetLogStr(err)))
		return
	}
	fmt.Println("建立链接成功...")
	return
}

```

### ③增删改查

```go
// 查询数据示例
func queryRowDemo() {
	sqlStr := "select * from user_tag where id > ?"
	rows, err := caller.MyDB.Query(sqlStr, 250)
	if err != nil {
		fmt.Printf("query failed, err:%v\n", err)
		return
	}
	defer rows.Close()

	for rows.Next() {
		var ut UserTag
		err := rows.Scan(&ut.Id, &ut.Name, &ut.EntityId, &ut.TimeCreated)
		if err != nil {
			fmt.Printf("scan failed, err:%v\n", err)
			return
		}
		fmt.Printf("res=%s\n", utils.GetLogStr(ut))
	}
}

// 插入数据
func insertRowDemo() {
	sqlStr := "insert into user_tag(name, entity_id, time_created) values (?,?,?)"
	nowTime := time.Now().Format("2006-01-02 15:04:05")
	ret, err := caller.MyDB.Exec(sqlStr, "王五", 2, nowTime)
	if err != nil {
		fmt.Printf("insert failed, err:%v\n", err)
		return
	}
	theID, err := ret.LastInsertId() // 新插入数据的id
	if err != nil {
		fmt.Printf("get lastinsert ID failed, err:%v\n", err)
		return
	}
	fmt.Printf("insert success, the id is %d.\n", theID)
}

// 更新数据
func updateRowDemo() {
	sqlStr := "update user_tag set time_created=? where id = ?"
	nowTime := time.Now().Format("2006-01-02 15:04:05")
	ret, err := caller.MyDB.Exec(sqlStr, nowTime, 259)
	if err != nil {
		fmt.Printf("update failed, err:%v\n", err)
		return
	}
	n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("get RowsAffected failed, err:%v\n", err)
		return
	}
	fmt.Printf("update success, affected rows:%d\n", n)
}

// 删除数据
func deleteRowDemo() {
	sqlStr := "delete from user_tag where id = ?"
	ret, err := caller.MyDB.Exec(sqlStr, 3)
	if err != nil {
		fmt.Printf("delete failed, err:%v\n", err)
		return
	}
	n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("get RowsAffected failed, err:%v\n", err)
		return
	}
	fmt.Printf("delete success, affected rows:%d\n", n)
}

```

## (2) io版orm框架 (！用这个)

官网： http://gorm.io/

### ①引入

```shell
go get gorm.io/driver/mysql
go get gorm.io/gorm
```



### ②初始化

```go
import (
	"fmt"

	"gorm.io/driver/mysql"
	"gorm.io/gorm"
)

var (
	MyDB *gorm.DB
)

func Init() {
	// 初始化数据库
	if err := InitDB(); err != nil {
		fmt.Println(fmt.Sprintf("database err, err=%v",err))
	}
}

func InitDB() (err error) {
	dsn := "root:991113@tcp(127.0.0.1:3306)/chuangyitest?charset=utf8&parseTime=True&loc=Local"
	MyDB, err = gorm.Open(mysql.Open(dsn), &gorm.Config{})
	if err != nil {
		fmt.Println(fmt.Sprintf("database open err, err=%v", err))
		return
	}
	fmt.Println("success...")
	return
}
```

### ③增删改查

说明：Debug()可以查看执行的sql语句。

```go
// 增
func createDemo() {
	userTag := UserTag{
		Name:     "java",
		EntityId: 2,
	}
	caller.MyDB.Debug().Table("user_tag").Create(&userTag)
    // 新增的主键
	fmt.Println(utils.GetLogStr(userTag.Id))
}

// 改
func updateDemo() {
	userTag := UserTag{
		Name: "玩耍2",
	}
	aa := caller.MyDB.Debug().Table("user_tag").Where("id=?", 259).Updates(&userTag) // 注意是Updates
	fmt.Println(utils.GetLogStr(aa))
}

// 查
func queryDemo() {
	userTagList := make([]UserTag, 0)
	caller.MyDB.Debug().Table("user_tag").Where("id = ?", 259).Find(&userTagList)
	fmt.Println(utils.GetLogStr(userTagList))
}

```

### ④gorm gen的使用

**a.先安装**(会安装到gopath的bin目录下，windows电脑，需要将该路径加入到系统路径)

```shell
go install gorm.io/gen/tools/gentool@latest
```

```shell
gentool -h

Usage of gentool:
 -c string
       config file path  配置文件路径
 -db string
       input mysql or postgres or sqlite or sqlserver. consult[https://gorm.io/docs/connecting_to_the_database.html] (default "mysql")
 -dsn string
       consult[https://gorm.io/docs/connecting_to_the_database.html]
 -fieldNullable
       generate with pointer when field is nullable
 -fieldWithIndexTag
       generate field with gorm index tag
 -fieldWithTypeTag
       generate field with gorm column type tag
 -modelPkgName string
       generated model code's package name  {生成结构体的路径}
 -outFile string
       query code file name, default: gen.go
 -outPath string
       specify a directory for output (default "./dao/query")   {生成query的路径}
 -tables string
       enter the required data table or leave it blank
 -onlyModel
       only generate models (without query file)
 -withUnitTest
       generate unit test for query code
 -fieldSignable
       detect integer field's unsigned type, adjust generated data type
```

eg :

```
--tables="orders"       # generate from `orders`

--tables="orders,users" # generate from `orders` and `users`

--tables=""             # generate from all tables 这样是全部的表名
```

**b.举例：**

说明1：windows电脑go install之后，把exe添加到系统路径，然后最好使用cmd运行, 先进入到项目目录，执行下面的命令(如果提示没有gcc命令，需要先安装该命令)

说明2：-modelPkgName属性是在-outPath路径的上一级目录的基础上的

```shell
gentool -dsn "root:991113@tcp(127.0.0.1:3306)/chuangyitest?charset=utf8&parseTime=True&loc=Local" -tables ""  -outPath "./dal/query" -modelPkgName "./models"
```

生成的项目结构为：

```shell
project
	dal
		models
		query
```



## (3) jinzhu版orm框架

官网： https://pkg.go.dev/github.com/jinzhu/gorm#Open

### ①引入

```shell
go get github.com/go-sql-driver/mysql
go get -u github.com/jinzhu/gorm
```

### ②初始化

```go
package caller

import (
	"fmt"

	_ "github.com/go-sql-driver/mysql"
	"github.com/jinzhu/gorm"
	"liuyaohui.lyh/Project03/utils"
)

var (
	MyDB *gorm.DB
)

func Init() {
	// 初始化数据库
	if err := InitDB(); err != nil {
		fmt.Println(fmt.Sprintf("database err, err=%s", utils.GetLogStr(err)))
	}
}

func InitDB() (err error) {
	MyDB, err = gorm.Open("mysql", "root:991113@tcp(127.0.0.1:3306)/chuangyitest?charset=utf8&parseTime=True&loc=Local")
	if err != nil {
		fmt.Println(fmt.Sprintf("database open err, err=%s", utils.GetLogStr(err)))
		return
	}
	fmt.Println("database success...")
	return
}
```

### ③增删改查

```go
// 增
func MyInsert() {
	caller.MyDB.Table("user_tag").Create(&UserTag{Name: "ik01001", EntityId: 2})
}

// 查
func myQuery() {
	var userTag UserTag
	caller.MyDB.Table("user_tag").First(&userTag, "id = ?", 259) // 查询code为259的userTag
	fmt.Printf("%s\n", utils.GetLogStr(userTag))

	userTagList := make([]UserTag, 0)
	caller.MyDB.Table("user_tag").Find(&userTagList, "id > ?", 257)
	fmt.Printf("%s\n", utils.GetLogStr(userTagList))
}

// 事务
func myTx() error {
	// 注意，一旦你在一个事务中，使用tx作为数据库句柄
	tx := caller.MyDB.Table("user_tag").Begin()
	// 注意：where一定要在更新操作前面,不然会数据全部更新
	// 方式1
	if err := tx.Where(map[string]interface{}{"id": 259}).Update(&UserTag{Name: "玩耍2"}).Error; err != nil {
		tx.Rollback()
		return err
	}
	// 方式2
	if err := tx.Where("id=?", 259).Update(&UserTag{Name: "玩耍333"}).Error; err != nil {
		tx.Rollback()
		return err
	}
	tx.Commit()
	return nil
}
```





# 2.redis

网址：https://liwenzhou.com/posts/Go/go_redis/

## (1)引入

```shell
go get -u github.com/go-redis/redis
```



## (2)初始化

### ①普通初始化

```go
// 声明一个全局的rdb变量
var rdb *redis.Client

// 初始化连接
func initClient() (err error) {
	rdb = redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "", // no password set
		DB:       0,  // use default DB
	})

	_, err = rdb.Ping().Result()
	if err != nil {
		return err
	}
	return nil
}
```

### ②v8初始化

```go
import (
	"context"
	"fmt"
	"time"

	"github.com/go-redis/redis/v8" // 注意导入的是新版本
)

var (
	rdb *redis.Client
)

// 初始化连接
func initClient() (err error) {
	rdb = redis.NewClient(&redis.Options{
		Addr:     "localhost:16379",
		Password: "",  // no password set
		DB:       0,   // use default DB
		PoolSize: 100, // 连接池大小
	})

	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	_, err = rdb.Ping(ctx).Result()
	return err
}
```

### ③get/set示例

```go
func redisExample() {
	err := rdb.Set("score", 100, 0).Err()
	if err != nil {
		fmt.Printf("set score failed, err:%v\n", err)
		return
	}

	val, err := rdb.Get("score").Result()
	if err != nil {
		fmt.Printf("get score failed, err:%v\n", err)
		return
	}
	fmt.Println("score", val)

	val2, err := rdb.Get("name").Result()
	if err == redis.Nil {
		fmt.Println("name does not exist")
	} else if err != nil {
		fmt.Printf("get name failed, err:%v\n", err)
		return
	} else {
		fmt.Println("name", val2)
	}
}
```



