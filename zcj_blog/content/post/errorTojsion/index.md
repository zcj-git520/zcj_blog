---
title: "go error类型转json"
date: 2021-10-09T22:00:38+08:00
draft: true
slug: Golang
image: 2.png
categories:
    - bug
    - Golang
---

## 问题

* 在收集服务的访问记录时，需要将访问记录保存，定义结构体如下

> 

```
type accessData struct {
	RemoteAddr string    // 远程访问主机地址
	RequestURI string    //访问的路由
	ServerName string    // 访问的服务名称
	AccessDate string    //访问的时间
	RunStatus bool       //服务是否正常运行
	RunError error       //运行报错：报错信息.
	ServerParam interface{} // 访问服务的参数
}
```

* 通过结构体转json，同时通过get请求得到图下结果
  ![](1.png)
* "RunError": {},被json转为{}的字符， 打印结构体，发现错误信息是有的：{192.168.1.101:53364 /v1/alarms/out/d GetOutAlarms 2021-10-12 10:09:42 false 没有这个报警🆔id <nil>},说明是error 转json问题
  
  ## 问题分析与解决
  
  * 问题分析查看error类型定义发现：error类型只是一个接口。它可以包含任何实现它的具体类型的值
  * 解决：将结构体中错误转化为字符串类型，同时用err.Error()返回是错误的字符串
  
  ```
  type accessData struct {
    RemoteAddr string    // 远程访问主机地址
    RequestURI string    //访问的路由
    ServerName string    // 访问的服务名称
    AccessDate string    //访问的时间
    RunStatus bool       //服务是否正常运行
    RunError string       //运行报错：报错信息.
    ServerParam interface{} // 访问服务的参数
    }
    type error interface {
    Error() string
    }
  ```
* 结果如图
* ![](2.png)