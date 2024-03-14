# GORM教程

## 概述

GORM 是目前比较热门的数据库ORM操作库，对开发者也很友好，使用非常方便简单。主要用于讲 strcut类型和数据库表记录进行映射，操作数据库的时候不需要手写SQL语句。这里主要介绍MySQL数据库。

- GORM 库的github地址：`https://github.com/go-gorm/gorm`

## 快速入门

- 安装依赖包
  - MYSQL驱动包：`go get -u gorm.io/driver/mysql`
  - GORM包：`go get -u gorm.io/gorm`

- 连接数据库

```golang
package main

import (
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
)

var DB *gorm.DB

func Createlink() *gorm.DB {
	dsn := "root:Dev1369143@tcp(127.0.0.1:3306)/test?charset=utf8mb4&parseTime=True&loc=Local"
	var err error
	DB, err = gorm.Open(mysql.Open(dsn), &gorm.Config{Logger: logger.Default.LogMode(logger.Info)})
	if err != nil {
		panic("连接数据库失败, error=" + err.Error())
	}
	return DB
}
```

- 创建数据库
```GOLANG

type User struct { // 默认创建表名字为users 结构体小写的复数形式， 可通过 TableName 自定义表名
	ID         int64     `gorm:"column:id"`
	Username   string    `gorm:"column:username"`
	Password   string    `gorm:"column:password"`
	CreateTime time.Time // 默认会创建字段为 create_time
}

func (User) TableName() string {
	return "t_user"
}

func CreateTable() {
	db := Createlink()
	if err := db.AutoMigrate(&User{}); err != nil {
		panic("表创建失败, error=" + err.Error())
	}
	fmt.Println("表创建成功")
}
```

```shell
mysql> desc t_user;
+-------------+-------------+------+-----+---------+----------------+
| Field       | Type        | Null | Key | Default | Extra          |
+-------------+-------------+------+-----+---------+----------------+
| id          | bigint      | NO   | PRI | NULL    | auto_increment |
| username    | longtext    | YES  |     | NULL    |                |
| password    | longtext    | YES  |     | NULL    |                |
| create_time | datetime(3) | YES  |     | NULL    |                |
+-------------+-------------+------+-----+---------+----------------+
```

- 插入数据

```
// 插入单条数据
func InsertData() {
	db := Createlink()

	tom := User{Username: "tom", Password: "tom123", CreateTime: time.Now()}

	if err := db.Create(&tom).Error; err != nil {
		panic("数据插入失败" + err.Error())
		return
	}
	fmt.Println("数据插入成功")
}

// 插入多条数据
func InsertManyData() {
	users := []User{
		{Username: "zhansan", Password: "zhansan123", CreateTime: time.Now()},
		{Username: "lisi", Password: "lisi123", CreateTime: time.Now()},
		{Username: "wangwu", Password: "wangwu123", CreateTime: time.Now()},
	}

	db := Createlink()
	db.Create(&users)
}
```
- 查询数据

```golang
// 查询并返回第一条数据
func QueryData() {
	var u User

	db := Createlink()
	
	result := db.Where("username=?", "tom").Find(&u)
	if errors.Is(result.Error, gorm.ErrRecordNotFound) {
		panic("找不到记录")
		return
	}
	fmt.Println(u.Username, u.Password)
}

// 查询并返回多条数据
func QueryManyData() {
	var users []User

	db := Createlink()

	db.Find(&users)
	for k, v := range users {
		fmt.Println(k, v)
	}
}
```


- 更新数据

```golang
// 更新单个字段，推荐使用
func UpdateData() {
	db := Createlink()

	db.Model(&User{}).Where("username=?", "tom").Update("password", "123456")
}

// Save 需要给全所有的字段
func SaveData() {

	db := Createlink()
	var u User
	db.Where("username=?", "zhanshan").Take(&u)
	u.Password = "123456"
	u.CreateTime = time.Now()
	db.Save(&u)
}
```

- 删除数据

```golang
// Save


func DeleteData() {
	db := Createlink()

	db.Where("username=?", "tom").Delete(&User{})
}
```

## GORM 模型定义

ORM框架定义数据库都需要预先定义模型，模型可以理解成数据模型，作为操作数据库的媒介。数据库查询出来的数据会先保存到预定义的模型对象，然后我们从模型对象中得到我们想要的数据 ，数据库插入数据也是先建立一个模型对象，然后通过模型对象讲数据保存到数据库中。在Golang中，gorm模型定义是通过struct实现的，这样我们就可以通过gorm库实现struct类型和mysql表数据的映射

gorm 标签语法： `gorm:"column:id[;unique]"`

| 标签名 |	说明
| --- | --- |
|column	|指定 db 列名|
|type	|列数据类型，推荐使用兼容性好的通用类型，例如：所有数据库都支持 bool、int、uint、float、string、time、bytes 并且可以和其他标签一起使用，例如：not null、size, autoIncrement… 像 varbinary(8) 这样指定数据库数据类型也是支持的。在使用指定数据库数据类型时，它需要是完整的数据库数据类型，如：MEDIUMINT UNSIGNED not NULL AUTO_INCREMENT
|serializer|	指定将数据序列化或反序列化到数据库中的序列化器, 例如: serializer:json/gob/unixtime
|size	|定义列数据类型的大小或长度，例如 size: 256
|primaryKey|	将列定义为主键
|unique|	将列定义为唯一键
|default|	定义列的默认值
|precision|	指定列的精度
|scale|	指定列大小
|not null|	指定列为 NOT NULL
|autoIncrement|	指定列为自动增长
|autoIncrementIncrement	|自动步长，控制连续记录之间的间隔
|embedded |	嵌套字段
|embeddedPrefix|	嵌入字段的列名前缀
|autoCreateTime|	创建时追踪当前时间，对于 int 字段，它会追踪时间戳秒数，您可以使用 nano/milli 来追踪纳秒、毫秒时间戳，例如：autoCreateTime:nano
|autoUpdateTime	|创建/更新时追踪当前时间，对于 int 字段，它会追踪时间戳秒数，您可以使用 nano/milli 来追踪纳秒、毫秒时间戳，例如：autoUpdateTime:milli
|index	|根据参数创建索引，多个字段使用相同的名称则创建复合索引，查看 索引 获取详情
|uniqueIndex|	与 index 相同，但创建的是唯一索引
|check	|创建检查约束，例如 check:age > 13，查看 约束 获取详情
|<-	|设置字段写入的权限， <-:create 只创建、<-:update 只更新、<-:false 无写入权限、<- 创建和更新权限
|->|	设置字段读的权限，->:false 无读权限
|-	|忽略该字段，- 表示无读写，-:migration 表示无迁移权限，-:all 表示无读写迁移权限
|comment|	迁移时为字段添加注释

## 关联查询

GROM 的关联查询又叫连表查询，常见的关联查询有一下两种
- `belongs To or has one `: 一对一关系,例如每个用户都用一张身份证,一个员工属于一个公司
- `has many`: 一对多关系，例如每个用户都有多张信用卡
- `many to many`： 多对多关系，例如一个班级可以有多个老师，多个老师可以属于某个班级

### Belongs to

belongs to 会与另一个模型建立了一对一的连接。 这种模型的每一个实例都 “属于” 另一个模型的一个实例。例如员工跟公司的对应关系，每个员工都属于一个公司

```golang
package main

import (
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
)

var DB *gorm.DB

func Createlink() *gorm.DB {
	dsn := "root:rootpwd@tcp(127.0.0.1:3306)/test?charset=utf8mb4&parseTime=True&loc=Local"
	var err error
	DB, err = gorm.Open(mysql.Open(dsn), &gorm.Config{Logger: logger.Default.LogMode(logger.Info)})
	if err != nil {
		panic("连接数据库失败, error=" + err.Error())
	}
	return DB
}

type Company struct {
	gorm.Model
	Name string
}
type User struct {
	gorm.Model
	Name      string
	CompanyID int
	Company   Company  # 每个员工都属于一个公司
}

func main() {
	db := Createlink()
	// 因为 User 跟 Company 关联了，所以迁移 User 会自动创建 Company 表
	db.AutoMigrate(&User{})

	jerry := User{
		Name: "jerry",
		Company: Company{
			Name: "Google",
		},
	}

	// 创建 jerry 用的的同事 Google 公司也会被自动创建出来
	db.Create(&jerry)
}
```
### Has Many 一对多

Has Many 用于建立一对多的模型，例如用户与信用卡，一个用户可以有多张信用卡

- 默认使用struct_id 作为 外键是
```golang
type CreditCard struct {
	gorm.Model
	Number string
	UserID uint
}
type User struct {
	gorm.Model
	CreditCards []CreditCard // `gorm:foreignKey:UserID`
}
```

- 重写外键
```golang
type User struct {
  gorm.Model
  CreditCards []CreditCard `gorm:"foreignKey:UserRefer"`
}

type CreditCard struct {
  gorm.Model
  Number    string
  UserRefer uint
}
```

- 重写引用
```golang
type User struct {
  gorm.Model
  MemberNumber string
  CreditCards  []CreditCard `gorm:"foreignKey:UserNumber;references:MemberNumber"`
}

type CreditCard struct {
  gorm.Model
  Number     string
  UserNumber string
}
```

### Many to Many 多对多

Many to Many 会在两个 model 中添加一张连接表。

```shell
// User 拥有并属于多种 language，`user_languages` 是连接表
type User struct {
  gorm.Model
  Languages []Language `gorm:"many2many:user_languages;"`
}

type Language struct {
  gorm.Model
  Name string
}
```



