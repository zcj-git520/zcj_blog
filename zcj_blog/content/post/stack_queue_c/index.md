---
title: "数据结构-栈(基于单项链表)&&队列(基于双向链表)c++实现"
date: 2021-12-31T21:00:08+08:00
draft: false
image: 1.jpg
categories:
    - language
    - C_C++
---
## 栈
基于单向链表实现
### 数据结构定义
```

template<class T>
class Stack
{
private:
    SinglyList<T> list;  // 存储数据的链表
public:
    Stack(/* args */){};
    ~Stack();
    // 获得栈的长度
    int len()
    {
        return list.getlen();
    }
    // 清空栈
    void clear();
    // 判断栈是否为空s
    bool isEmpty();
    // 将数据放入栈顶
    void push(T data);
    // 获取栈顶数据，并删除数据
    bool pop(T *info);
    // 获取栈顶数据但不删除数据
    bool getTopValue(T *info);
    // 显示所有栈的数据
    void showStack();
};
```

###  数据操作
```

template<class T>
void Stack<T> ::clear()
{
    list.clear();
}

template<class T>
void Stack<T> ::push(T data)
{
    list.inseartToHead(data);
}

template<class T>
bool Stack<T> ::pop(T *info)
{
    // 判断栈顶是否有值
    if(getTopValue(info))
    {
        list.deleteToHead();  // 删除栈顶值
        return true;
    }
    return false;
}

template<class T>
bool Stack<T> ::getTopValue(T *info)
{
     // 判断栈顶是否有值
    if(list.queryIndex(0, info))
    {
        return true;
    }
    return false;
}

template<class T>
void Stack<T> ::showStack()
{
    list.queryAll();
}
```
### 测试
```
int main()
{
    Stack<int> s;
    s.push(0);
    s.push(1);
    s.push(2);
    s.push(3);
    s.push(4);
    s.showStack();
    cout << "********************************" << endl;
    int info = -1;
     if(s.getTopValue(&info))
    {
        cout << "top value is: " <<info << endl;
    }
    if(s.getTopValue(&info))
    {
        cout << "top value is: " <<info << endl;
    }
    if(s.getTopValue(&info))
    {
        cout << "top value is: " <<info << endl;
    }
    cout << "********************************" << endl;
    s.showStack();
    cout << "********************************" << endl;
    if(s.pop(&info))
    {
        cout << "top value is: " <<info << endl;
    }
    if(s.pop(&info))
    {
        cout << "top value is: " <<info << endl;
    }
    if(s.pop(&info))
    {
        cout << "top value is: " <<info << endl;
    }
    if(s.pop(&info))
    {
        cout << "top value is: " <<info << endl;
    }
    cout << "********************************" << endl;
    s.showStack();
    cout << "********************************" << endl;
    system("pause");
    return 0;
}

```
## 队列
* 基于双向链表实现
### 数据结构定义
```

template<class T>
class Queue
{
private:
    TwoWayList<T> list;  // 定义的双向链表
public:
    Queue(/* args */){};
    ~Queue();
     // 获得队列的长度
    int len()
    {
        return list.getlen();
    }
    // 清空队列
    void clear();
    // 判断栈是否为空s
    bool isEmpty();
    // 将数据存入队列尾中
    void enQueue(T data);
    // 获取获得队列数据，并删除数据
    bool deQueue(T *info);
    // 获取获得队列数据，但不删除数据
    bool getQueueValue(T *info);
    // 显示所有队列的数据
    void showQueue();
};

```
### 数据的操作
```

template<class T>
void Queue<T> ::clear()
{
    list.clear();
}

template<class T>
bool Queue<T> ::isEmpty()
{
    return list.isEmpty();
}

template<class T>
void Queue<T> ::enQueue(T data)
{
    list.inseartTotail(data);
}

template<class T>
bool Queue<T> ::getQueueValue(T *info)
{
    if(list.queryIndex(0, info))
    {
        return true;
    }
    return false;
}

template<class T>
bool Queue<T> ::deQueue(T *info)
{
    if(list.queryIndex(0, info))
    {
        list.deleteToHead();
        return true;
    }
    return false;
}

template<class T>
void Queue<T> ::showQueue()
{
    list.queryAll();
}
```
### 测试
```
int main()
{
    Queue<int> q;
    q.enQueue(0);
    q.enQueue(1);
    q.enQueue(2);
    q.enQueue(3);
    q.enQueue(4);
    q.enQueue(5);
    q.showQueue();
    cout << "**************************************"<< endl;
    int info = -1;
    if(q.getQueueValue(&info))
    {
        cout << "queue top value is: " <<info << endl;
    }
    if(q.getQueueValue(&info))
    {
        cout << "queue top value is: " <<info << endl;
    }
    if(q.getQueueValue(&info))
    {
        cout << "queue top value is: " <<info << endl;
    }
    q.showQueue();
    cout << "**************************************"<< endl;
    if(q.deQueue(&info))
    {
        cout << "queue top value is: " <<info << endl;
    }
    q.showQueue();
    cout << "**************************************"<< endl;
    if(q.deQueue(&info))
    {
        cout << "queue top value is: " <<info << endl;
    }
    q.showQueue();
    cout << "**************************************"<< endl;
    if(q.deQueue(&info))
    {
        cout << "queue top value is: " <<info << endl;
    }
    q.showQueue();
    cout << "**************************************"<< endl;
    q.clear();
    q.showQueue();
    system("pause");
    return 0;
}

```
## 源码
* [我的github:https:https://github.com/zcj-git520/DataStructuresAlgorithmsForC/tree/master/stack_queue/queue](https://github.com/zcj-git520/DataStructuresAlgorithmsForC/tree/master/stack_queue/queue)
* [我的github:https:https://github.com/zcj-git520/DataStructuresAlgorithmsForGo/tree/master/stack_queue/stackhttps://github.com/zcj-git520/DataStructuresAlgorithmsForC/tree/master/stack_queue/stack](https://github.com/zcj-git520/DataStructuresAlgorithmsForC/tree/master/stack_queue/stack)

