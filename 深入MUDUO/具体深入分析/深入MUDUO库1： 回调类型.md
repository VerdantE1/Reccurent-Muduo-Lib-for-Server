# 步步分析Muduo .1 简要看看回调

首先我们简要分析TcpServer类。

muduo::net::TcpServer



##### TcpServer里面的回调函数

TcpServer里面存着一系列的回调函数指针

![image-20221030143535941](assets/image-20221030143535941.png)

这个回调函数指针通过类似下面的接口设置

![image-20221030143407134](assets/image-20221030143407134.png)



我们进入到其中一个来看到底是什么类型

进入里面

![image-20221030143755353](assets/image-20221030143755353.png)

```cpp
typedef std::shared_ptr<TcpConnection> TcpConnectionPtr;
typedef std::function<void()> TimerCallback;
typedef std::function<void (const TcpConnectionPtr&)> ConnectionCallback;
typedef std::function<void (const TcpConnectionPtr&)> CloseCallback;
typedef std::function<void (const TcpConnectionPtr&)> WriteCompleteCallback;
typedef std::function<void (const TcpConnectionPtr&, size_t)> HighWaterMarkCallback;

// the data has been read to (buf, len)
typedef std::function<void (const TcpConnectionPtr&,Buffer*,Timestamp)> MessageCallback;
```



可以看到每个类都是一个模板函数function的实例，其类型都是返回void ,参数均是TcpConnectionPtr的指针类。

从TcpConnectionPtr名字上看是一个智能指针