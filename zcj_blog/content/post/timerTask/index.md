---
title: "timerTask 定时任务包"
date: 2021-12-15T22:00:08+08:00
draft: false
image: 1.png
categories:
    - language
    - golang
    - bug
---

## timerTask
定时任务：可以定时执行单个或者对个任务：
 1. 定时定时任务
 2. 停止定时任务
 3. 重置定时时间，并执行定时任务
 4. 定时开启定时任务
 5. 定时结束定时任务  
 
## timerTask结构

```
type timerConfig struct {
	timing       time.Duration      // 定时时间
	startTiming  time.Duration      // 定时开启定时任务时间
	stopTiming   time.Duration      // 定时停止定时任务时间
	running   	 bool               // 定时器运行的状态,运作中为true/没有运行为false
	tasks        []func()           // 定时任务
	stopChan     chan struct{}      // 停止定时器日任务
	runningMu    sync.Mutex
	Waiter       sync.WaitGroup
}
```

## 定时任务的开始和结束
```
函数原型：NewTimerTask(d time.Duration, tasks []func(), opts ... Option) *timerConfig
参数为：统一的定时任务时间,定时任务集, 可选参数【定时开启定时任务时间(SetTimerStart(d time.Duration)), 定时停止定时任务时间(SetTimerStop(d time.Duration))】
Start()开始定时任务, Stop()结束定时任务 

```
使用
```
t1 := 1 * time.Second
	tasks := []func(){printTest1, printTest2, printTest3, printTest4, printTest5}
	timer := NewTimerTask(t1, tasks)
	timer.Waiter.Add(2)
	timer.start()
	go func() {
		select {
		case <-time.After(5*time.Second):
			timer.stop()
			timer.Waiter.Done()
			return
		}

	}()
	timer.Waiter.Wait()
```

## 重置定时时间任务

```
函数原型：Reset(d time.Duration) // 重置定时的时间
```
使用：
```
	t1 := 1 * time.Second
	tasks := []func(){printTest1, printTest2, printTest3, printTest4, printTest5}
	timer := NewTimerTask(t1, tasks)
	timer.Waiter.Add(3)
	timer.start()
	go func() {
		select {
		case <-time.After(10*time.Second):
			timer.stop()
			timer.Waiter.Done()
			return
		}

	}()
	go func() {
		select {
		case <-time.After(5*time.Second):
			timer.Reset(2*time.Second)
			timer.Waiter.Done()
			return
		}

	}()
	timer.Waiter.Wait()
```

## 定时开始定时任务
```
函数原型：TimerStart() ,需要在初始化时,设定定时开始定时任务的时间
```
使用：
```
	t1 := 1 * time.Second
	ts := time.Now()
	tasks := []func(){printTest1, printTest2, printTest3, printTest4, printTest5}
	timer := NewTimerTask(t1, tasks, SetTimerStart(3*time.Second))
	timer.TimerStart()
	t.Logf("定时时间为：%v", time.Now().Sub(ts))
	timer.Waiter.Add(2)
	timer.start()
	go func() {
		select {
		case <-time.After(5*time.Second):
			timer.stop()
			timer.Waiter.Done()
			return
		}

	}()
	timer.Waiter.Wait()
```

## 定时结束定时任务
```
函数原型：TimerStop(),设定定时结束定时任务的时间
```
* 使用：
```
t1 := 1 * time.Second
	tasks := []func(){printTest1, printTest2, printTest3, printTest4, printTest5}
	timer := NewTimerTask(t1, tasks, SetTimerStop(3*time.Second))
	timer.Waiter.Add(1)
	timer.start()
	time.Sleep(1*time.Second)
	ts := time.Now()
	timer.TimerStop()
	t.Logf("定时时间为：%v", time.Now().Sub(ts))
	timer.Waiter.Wait()
```
## 源码
* [我的github:https://github.com/zcj-git520/timerTask](https://github.com/zcj-git520/timerTask)

