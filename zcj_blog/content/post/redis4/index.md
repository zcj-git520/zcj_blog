---
title: "redis GO语言实践一"
date: 2022-07-08T22:00:38+08:00
draft: false
image: 1.png
categories:
    - language
    - golang
---

## redis GO语言实践一

### 简单介绍
- 使用go 语言操作redis数据库
- 使用的库为：go get -u github.com/go-redis/redis
- 对redis数据库的连接，对字符串和列表的操作

### 代码如下

```go
package main

import (
	"fmt"
	"github.com/go-redis/redis"
	"log"
	"time"
)

const (
	ADDR = "192.168.1.128:6379"
	PASSWORD = ""
	DB = 0
)

type RedisInfo struct {
	Option    *redis.Options
	RedisDb   *redis.Client
}


// 连接redis数据库
func (r *RedisInfo)ConnRedis() error{
	redisDb := redis.NewClient(r.Option)
	// 判断是否成功连接数据库
	_, err := redisDb.Ping().Result()
	if err != nil {
		return err
	}
	r.RedisDb = redisDb
	return nil
}

/* 操作字符串类型 */
// 操作set/get
func (r *RedisInfo)OperaStringSGet()  {
	// 添加key/value 可设置过期时间
	err := r.RedisDb.Set("Key1", "value1", 4*time.Millisecond).Err()
	if err != nil {
		fmt.Println("添加key失败", err)
		return
	}
	time.Sleep(4*time.Millisecond)
	// 获取key的值
	value, err := r.RedisDb.Get("Key1").Result()
	if err != nil {
		fmt.Println("获取key的值失败", err)
		return
	}
	if value == ""{
		fmt.Println("key 不存在")
		// 如果不存在则就添加, 不设置过期时间
		r.RedisDb.SetNX("Key1", "setnx value", 0)
	}else {
		fmt.Printf("获得value：%v \n", value)
	}
}

// 操作mset/mget
func (r *RedisInfo)OperaStringMSGet()  {
	// 批量添加key-value
	err := r.RedisDb.MSet("key0", "value0", "key2", "value2", "key3", "value3","key4", "value4").Err()
	if err != nil {
		fmt.Printf("批量添加key-value 失败 %v \n", err)
		return
	}
	// 批量获取value
	values, err := r.RedisDb.MGet("key0","key1","key2","key3", "key4").Result()
	if err != nil {
		fmt.Printf("批量获取value 失败：%v \n", err)
		return
	}
	fmt.Printf("values is : %v\n", values)
	// 删除keys
	err = r.RedisDb.Del("key0").Err()
	if err != nil {
		fmt.Printf("删除keys 失败 %v\n", err)
		return
	}
	// 获得指定的keys
	keys, err := r.RedisDb.Keys("k*").Result()
	if err != nil {
		fmt.Printf(" 获得指定的keys 失败 %v\n", err)
	}else{
		fmt.Printf("keys is :%v\n", keys)
	}
}

// 操作自增自减
func (r *RedisInfo)OperaStringNcr(){
	// 增加数据
	err := r.RedisDb.Set("score", 120, 0).Err()
	if err != nil {
		fmt.Println("增加数据失败", err)
		return
	}
	//自增操作
	err = r.RedisDb.Incr("score").Err()
	if err != nil {
		fmt.Println("自增失败", err)
		return
	}
	value, _ := r.RedisDb.Get("score").Result()
	fmt.Printf("成绩为:%v\n", value)
	// 步长增加
	err = r.RedisDb.IncrBy("score", 10).Err()
	if err != nil {
		fmt.Println("步长增加失败", err)
		return
	}
	value, _ = r.RedisDb.Get("score").Result()
	fmt.Printf("成绩为:%v\n", value)
	// 自减
	err = r.RedisDb.Decr("score").Err()
	if err != nil {
		fmt.Println("自减失败", err)
		return
	}
	value, _ = r.RedisDb.Get("score").Result()
	fmt.Printf("成绩为:%v\n", value)
	// 步长减少
	err = r.RedisDb.DecrBy("score", -20).Err()
	if err != nil {
		fmt.Println("步长减少失败", err)
		return
	}
	value, _ = r.RedisDb.Get("score").Result()
	fmt.Printf("成绩为:%v\n", value)
	// 浮点数的自增/自减
	err = r.RedisDb.IncrByFloat("score", -20.5).Err()
	if err != nil {
		fmt.Println("浮点数的自增/自减失败", err)
		return
	}
	value, _ = r.RedisDb.Get("score").Result()
	fmt.Printf("成绩为:%v\n", value)
}

/* list类型的操作*/
//
func (r *RedisInfo)OperaListLData() {
	// 左边Push任意增加key-value
	err := r.RedisDb.LPush("Lkey1", "Lvalue1", "Lvalue2", "Lvalue3").Err()
	if err != nil {
		fmt.Println("左边Push增加key失败：", err)
		return
	}
	// 左边增加数据，当list不存在时，不能增加
	err = r.RedisDb.LPushX("LXkey", "LXValue").Err()
	if err != nil {
		fmt.Println("左边Push增加key失败：", err)
		return
	}
	// 获得list长度
	LLen, err := r.RedisDb.LLen("Lkey1").Result()
	if err != nil {
		fmt.Println("获得list长度失败：", err)
		return
	}
	fmt.Println("Lkey1 list len is :", LLen)
	// 获取列表数据
	LData, err := r.RedisDb.LRange("Lkey1", 0, -1).Result()
	if err != nil {
		fmt.Println("获取列表数据失败", err)
		return
	}
	fmt.Println("列表数据为：", LData)
	// 修改list中存在的value
	err = r.RedisDb.LSet("Lkey1", 1, "updateValue").Err()
	if err != nil {
		fmt.Println("修改值失败", err)
		return
	}
	// 移除并返回list左边第一个数据
	PData, err := r.RedisDb.LPop("Lkey1").Result()
	if err != nil {
		fmt.Println("移除数据失败", err)
		return
	}
	fmt.Println("PData is :", PData)
	// 移除list中重复的value,count为1时，移除一个
	count, err := r.RedisDb.LRem("Lkey1", 2,"Lvalue1").Result()
	if err != nil {
		fmt.Println("移除多个重读的value值失败", err)
		return
	}
	fmt.Println("count is", count)
	// 列表索引获得数据
	data, err := r.RedisDb.LIndex("Lkey1", 1).Result()
	if err != nil {
		fmt.Println("获得数据失败", err)
		return
	}
	fmt.Println("data is ", data)
	// 将某个数据插入到某个值前(before)插入到某个值之后(after)
	err = r.RedisDb.LInsert("Lkey1", "before", "updateValue", "insetData").Err()
	//err := r.RedisDb.LInsertBefore("Lkey1", "updateValue", "insetData").Err()
	if err != nil {
		fmt.Println("插入数据失败", err)
		return
	}
	// 截取名称为key的list
	ok, err := r.RedisDb.LTrim("Lkey1", 1,3).Result()
	if err != nil {
		fmt.Println("获取新的list失败")
		return
	}
	fmt.Println( ok)
	LData, err = r.RedisDb.LRange("Lkey1", 0, -1).Result()
	if err != nil {
		fmt.Println("获取列表数据失败", err)
		return
	}
	fmt.Println("列表数据为：", LData)
}

func main() {
	opt := &redis.Options{
		Network:            "tcp",    // 网络类型 tcp 或者 unix.
		Addr:               ADDR,
		OnConnect:          nil,      // 新建一个redis连接的时候，会回调这个函数
		Password:           PASSWORD, // redis数据库连接的密码
		DB:                 DB,       // redis数据库，序号从0开始，默认是0，可以不用设置
		MaxRetries:         5,        //  redis操作失败最大重试次数，默认不重试。
		/*
			MinRetryBackoff:    0,        // 最小重试时间间隔  默认是 8ms ; -1 表示关闭
			MaxRetryBackoff:    0,        // 最小重试时间间隔  默认是 512ms ; -1 表示关闭
			DialTimeout:        0,        // redis连接超时时间. 默认为5秒
			ReadTimeout:        0,        // socket读取超时时间 默认时间为3秒
			WriteTimeout:       0,        // socket写超时时间
			PoolSize:           0,        // redis连接池的最大连接数.默认连接池大小等于 cpu个数 * 10
			MinIdleConns:       0,        //redis连接池最小空闲连接数.
			MaxConnAge:         0,        // redis连接最大的存活时间，默认不会关闭过时的连接.
			PoolTimeout:        0,        // 当你从redis连接池获取一个连接之后，连接池最多等待这个拿出去的连接多长时间。  默认是等待 ReadTimeout + 1 秒.
			IdleTimeout:        0,        // redis连接池多久会关闭一个空闲连接. 默认是 5 分钟. -1 则表示关闭这个配置项
			IdleCheckFrequency: 0,        // 多长时间检测一下，空闲连接  默认是 1 分钟. -1 表示关闭空闲连接检测
			TLSConfig:          nil,      // 要使用的TLS配置。设置TLS时将进行协商
		*/
	}
	redisClient := &RedisInfo{
		Option:  opt,
		RedisDb: nil,
	}
	err := redisClient.ConnRedis()
	if err != nil {
		log.Fatal("redis 数据库连接失败！！！", err)
	}
	fmt.Println("数据库连接成功")
	redisClient.OperaStringSGet()
	redisClient.OperaStringMSGet()
	redisClient.OperaStringNcr()
	redisClient.OperaListLData()
}
```

