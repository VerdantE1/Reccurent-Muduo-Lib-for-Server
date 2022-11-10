# TCP网络编程本质

如MUDUO库，大部分网络IO操作都已经封装，暴露给我们的只有三个半，这三个半也就是TCP网络编程的本质

## 三个半步骤

1.连接的建立 处理。

2.连接的断开 处理。

3.消息到达（读） 处理

3.5消息发送完毕（写） 处理

**我们只用写该干什么即怎么干，而不用去监听各种事件。各种事件的响应由网络库帮我们监听。**



![image-20221030141534637](assets/image-20221030141534637.png)

在这个图里面，在框架的作用下，我们只用写EventHandler和Event并将Event加入到Reactor.Dumliplex和Reactor是由框架维护的。







## MUDUO库封装了什么？

很显然，封装了绑定服务器过程，但是当连接accpet时要做两个事情将cfd上epoll树管理，再者是业务逻辑，前面的上树都是由muduo库维护。 其次启动反应堆，监听任何一个连接，任何一个读写事件都是由库去维护，我们只需要指定库要做什么，每次有一个事件时muduo库会自动去做这个事情。我们要分几个线程去做？即如何处理这个监听器？依然是库维护，我们只需要指定即可，库默认以1：n处理lfd与cfd。
所以muduo完完全全的让业务逻辑和连接底层分开，我们用户只需要告诉muduo库要做什么即可。



##### 我们要做啥呢？

创建一个服务器对象，创建一个反应堆，写回调函数，告诉服务器对象IP地址与要维护的反应堆，启动服务器对象(初始化服务器对象，将reactor等都注册到服务器上去），启动反应堆（epoll_wait,以阻塞方式等待新的用户连接，已经连接用户的读写事件）。





##### 代码架构

```cpp
#include <muduo/net/TcpServer.h>
#include <muduo/base/Logging.h>
#include <muduo/net/EventLoop.h>
#include <iostream>
#include <functional>


using namespace std;
using namespace muduo;
using namespace muduo::net;
using namespace placeholders;


//1.构建服务器对象
//2.创建EventLoop事件循环对象(反应堆对象)的指针
//3.明确TcpServer构造函数需要什么参数，输出EchoServer的构造函数
//4.在当前服务器类的构造函数中，注册处理连接的回调函数处理
//5.设置是否使用多线程，默认单线程。多线程适应：1:N


class EchoServer
{
public:
    EchoServer(EventLoop * mainreactor,
                const InetAddress & LfdAddr,
                const string & nameArg)
            :_server(mainreactor,LfdAddr,nameArg),_loop(mainreactor)
            {
                
                
                //给服务器注册用户连接的创建和断开回调
                _server.setConnectionCallback(std::bind(&EchoServer::onConnect,this,_1));


                //给服务器注册用户读写事件回调
                _server.setMessageCallback(std::bind(&EchoServer::onMessage,this,_1,_2,_3));

                //设置多少线程
                _server.setThreadNum(2);

                
                
            }

    void start()
    {
        _server.start();
    }
  


private:
    void onConnect(const TcpConnectionPtr& coon){
        //连接回调业务逻辑
        if(coon->connected())
        {
            cout <<  coon -> peerAddress().toIpPort()  << " -> " << coon->localAddress().toIpPort();
            cout << "state online" <<endl;
        }
        else{
            cout <<  coon -> peerAddress().toIpPort()  << " -> " << coon->localAddress().toIpPort();
            cout << "state offline" <<endl;
            

        }

    }

    void onMessage(const TcpConnectionPtr& coon,Buffer* buffer,Timestamp time){
        //有消息的业务逻辑
        string  buf = buffer->retrieveAllAsString();
        cout << "recv data" << buf  << "time:" << time.toString()<<endl; 
        coon->send(buf);
    }


    TcpServer _server;
    EventLoop * _loop; // main reactor


    
};


int main()
{
    EventLoop loop;
    InetAddress addr("127.0.0.1",6000);
    EchoServer server(&loop,addr,"EchoServer");

    server.start();  //初始化上epoll树
    loop.loop();

    return 0;
}
```



