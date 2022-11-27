# EventLoopThreadPool类

一个线程有线程栈，一个函数有一个栈帧。malloc是向OS申请内存资源，这个是所有线程共享（找得到的）的。而线程栈如果不明确给出地址，彼此线程是找不到的。



EventLoopThreadPool 是管理每个EventLoopThread的池式结构

##### 工作的事情：

1.创建新eventloop与thread。 （绑定工作由EventLoopThread进行）

2.维护所有EventLoop（当该对象销毁，调用里面的智能指针将销毁所有EventLoopThread,即调用所有的~EventLoopThread  【Thread需要回收，loop是栈帧里面创建会自动销毁这是属于~EventLoopThread的】)，







##### 说明：

1.一个EventLoopThreadPool创建必有一个main Loop (main Reactor)。故线程数量numThreads代表的是subLoop的数量而不是总loop的数量。



2.很显然baseloop并不在loops中，loops中存储的subloop,但是baseloop在getallloop()轮询中是要返回的。