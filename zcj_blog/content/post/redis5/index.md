---
title: "redis GO语言实践二"
date: 2022-07-15T22:00:38+08:00
draft: false
image: 1.png
categories:
    - language
    - golang
---

## redis GO语言实践二

### 简单介绍
- 使用go 语言操作redis数据库
- 使用的库为：go get -u github.com/go-redis/redis
- 对redis数据库的map，集合和有序集合的操作

### 代码如下

```go
/*操作hash Map数据类型*/
func (r *RedisInfo)OperaHashMap(){
	//添加单个数据
	err := r.RedisDb.HSet("hKey","filed0", "value0").Err()
	if err != nil {
		fmt.Println("添加单个数据失败", err)
		return
	}
	// 批量添加数据
	hData := make(map[string]interface{})
	hData["filed1"] = "value1"
	hData["filed2"] = "value2"
	hData["filed3"] = "value3"
	hData["filed4"] = 20
	hData["filed5"] = 12.5
	err = r.RedisDb.HMSet("hKey", hData).Err()
	if err != nil {
		fmt.Println("批量添加数据失败",err)
		return
	}
	// 单个获取数据
	value, err := r.RedisDb.HGet("hKey", "filed0").Result()
	if err != nil {
		fmt.Println("单个获取数失败", err)
		return
	}
	fmt.Println("value is ", value)
	// 整数数据的自增
	err = r.RedisDb.HIncrBy("hKey", "filed4", 50 ).Err()
	if err != nil {
		fmt.Println("整数数据的自增失败", err)
		return
	}
	// 浮点数数据的自增
	err = r.RedisDb.HIncrByFloat("hKey", "filed5", 52.64).Err()
	if err != nil {
		fmt.Println("浮点数数据的自增失败", err)
		return
	}
	// 获取所有字段名
	keys, _:= r.RedisDb.HKeys("hKey").Result()
	fmt.Println("keys is ", keys)
	// 获取字段数量
	kLen, _ := r.RedisDb.HLen("hKey").Result()
	fmt.Println("kLen is ", kLen)
	// 批量获取数据
	values, err := r.RedisDb.HMGet("hKey", "filed1", "filed2","filed3").Result()
	if err != nil {
		fmt.Println("批量获取数据失败", err)
		return
	}
	fmt.Println("values is ", values)
	// 删除数据
	err = r.RedisDb.HDel("hKey", "filed1", "filed0").Err()
	if err != nil {
		fmt.Println("删除数据失败",err)
		return
	}
	// 判断数据是否存在
	ok, _ := r.RedisDb.HExists("hKey", "filed1").Result()
	if ok{
		fmt.Println("filed1 is 存在的")
	}else{
		fmt.Println("filed1 is 不存在的")
	}
	// 获取所有数据
	allValues, err := r.RedisDb.HGetAll("hKey").Result()
	if err != nil {
		fmt.Println("获取所有数据", err)
		return
	}
	fmt.Println("allValues is ", allValues)

}

/*操作set集合*/
func (r *RedisInfo) OperaSet() {
	// 添加数据
	err := r.RedisDb.SAdd("sKey", "test1","value1", 10, 25,3.215).Err()
	err = r.RedisDb.SAdd("sKey1", "value1", 110, 25,3.215, "ff","zvj",25).Err()
	if err != nil {
		fmt.Println("添加数据失败", err)
		return
	}
	// 获取数据个数
	cLen, _ := r.RedisDb.SCard("sKey").Result()
	fmt.Println("cLen is ", cLen)
	// 删除第一个数据
	fData, _ := r.RedisDb.SPop("sKey").Result()
	fmt.Println("第一个数据为：", fData)
	// 删除某个元素，并返回集合数量
	cLen, _ = r.RedisDb.SRem("sKey", "value1").Result()
	fmt.Println("删除后cLen is ", cLen)
	// 判断某个元素是否存在
	ok, _ := r.RedisDb.SIsMember("sKey", "value1").Result()
	if ok{
		fmt.Println("value1 is 存在")
	}else {
		fmt.Println("value1 is 不存在")
	}
	// 获取所有集合数据
	sDataS, _ := r.RedisDb.SMembers("sKey").Result()
	fmt.Println("sDataS is ", sDataS)
	// 求交集并保存在某个集合中
	dataLen, _ := r.RedisDb.SInterStore("sInterStore", "sKey", "sKey1").Result()
	fmt.Println("交集数据个数为：", dataLen)
	// 求差集并保存在某个集合中
	dataLen, _ = r.RedisDb.SDiffStore("sDiffStore", "sKey", "sKey1").Result()
	fmt.Println("差集数据个数为：", dataLen)
	// 求并集并保存在某个集合中
	dataLen, _ = r.RedisDb.SUnionStore("sUnionStore", "sKey", "sKey1").Result()
	fmt.Println("并集数据个数为：", dataLen)

}

/*操作有序集合*/
func (r *RedisInfo) OperaSortSet() {
	// 添加数据
	languages := [] redis.Z{
		{Score: 90.0, Member: "Golang"},
		{Score: 98.0, Member: "Java"},
		{Score: 95.0, Member: "Python"},
		{Score: 97.0, Member: "JavaScript"},
		{Score: 92.0, Member: "C/C++"},
	}
	num, err := r.RedisDb.ZAdd("zKey", languages...).Result()
	if err != nil {
		fmt.Println("添加数据", err)
		return
	}
	fmt.Println("add 数据给个数为", num)
	// 增加分时
	err = r.RedisDb.ZIncrBy("zKey", 3, "Golang").Err()
	if err != nil {
		fmt.Println("增分数失败", err)
		return
	}
	// 返回数据个数
	zLen, _ := r.RedisDb.ZCard("zKey").Result()
	fmt.Println("zLen is ", zLen)
	// 统计分数段的元素个数
	zLen, _ = r.RedisDb.ZCount("zKey", "0", "150").Result()
	fmt.Println("count zLen is ", zLen)
	// 排序
	value, _ := r.RedisDb.ZRange("zKey", 0,-1).Result()
	fmt.Println("value is ", value)
	// 删除某个元素
	r.RedisDb.ZRem("zKey", "JavaScript")
}
```

