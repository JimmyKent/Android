Android要求我们在主线程更新UI，如果在非主线程更新UI，就出出现如下异常：

android.view.ViewRootImpl$CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views.
at android.view.ViewRootImpl.checkThread(ViewRootImpl.java:6462)  
进入ViewRootImpl.checkThead()源码:  

```
void checkThread() {  
	if (mThread != Thread.currentThread()) {  
		throw new CalledFromWrongThreadException(  
			"Only the original thread that created a view hierarchy can touch its views.");  
	}  
}  
```

Android UI更新的时候在这里进行的线程判断，如果不是主线程，就不能更新UI，即抛出异常

CalledFromWrongThreadException。


Handler能够让你发送和处理消息，以及Runnable对象；每个Handler对象对应一个Thread和Thread的消息队列。当你创建一个Handler时，它就和Thread的消息队列绑定在一起，然后就可以传递消息和runnable对象到消息队列中，执行消息后就从消息队列中退出。  

Handler的作用就是：调度消息和runnable对象去被执行；使动作在不同的线程中被执行。

　　当一个应用程序中进程被创建时，它的主线程专门运行消息队列（messageQueue），去管

理顶层的应用程序相关的对象如：activity，broadcastReceiver，windows等，你可以创建你

的Thread，和主线程进行交互——通过Handler，交互的方法就是通过post或者sendMessage。

但是在你的新线程中，给定的Message或者Runnable，会在适当的时候的被调度和处理。

（即不会被立即处理——阻塞式）。

——建立消息处理模型/系统。


从一般的消息系统模型的建立大致构成以下几个部分：

　　　　l  消息原型

　　　　l  消息队列

　　　　l  发送消息

　　　　l  消息循环

　　　　l  消息获取

　　　　l  消息派发

　　　　l  消息处理



消息系统模型一般会包括以上七个部分（消息原型，消息队列，消息发送，消息循环，消息获取，

消息派发，消息处理）。实际上的核心是消息队列和消息循环，其余部分都是围绕这两部分进行的。

　　从前面文档的分析中我们知道Handler就是用来建立消息处理的系统模型，那么和这里基本消息

系统模型相比，那么Handler又是如何囊括这七个部分的呢？

　　在Android中对这六个部分进行了抽象成四个独立的部分：

　　　　Handler，Message，MessageQueue，Looper；

　　Message就是消息原型，包含消息描述和数据，
　　MessageQueue就是消息队列，
　　Looper完成消息循环
　　Handler就是驾驭整个消息系统模型，统领Message，MessgeQueue和Looper；



```
ThreadLocal 对应每个线程的Looper唯一,如果不用ThreadLocal,则需要用HashMap之类的数据结构.  
Looper和Handler的同步关系:如果HandlerThread建立,而Looper为空,则该HandlerThread 进入wait,直到执行了该Thread的run函数,Looper.myLooper之后,notifyAll唤醒.
MessageQueue执行enqueueMessage之后,执行native方法,nativeWake,这里之后是不是最后调用loop唤醒?  
Handler可以有多个, 可以动不动就new出来,推荐使用一个,这样就只有一个队列了
```


```
HandlerThread实现:
在run的时候:
Looper.prepare(); //利用ThreadLocal创建线程唯一副本
synchronized (this) {
    mLooper = Looper.myLooper(); //得到当前线程的Looper
    notifyAll(); //这个时候Looper准备好了,唤醒其他调用线程的地方
}
Process.setThreadPriority(mPriority);
onLooperPrepared();
Looper.loop(); //开始循环


在getHandler时,调用 getLooper(),在getLooper的时候, 如果线程启动后，Looper 还没创建，就 wait() 等待 创建 Looper 后 notify, 在线程run的时候,会notifyAll唤醒.
```

onLooperPrepared空实现,可以在子类中重写 onLooperPrepared，做一些初始化工作

通过MessageQueue.java 和底层的 MessageQueue.cpp通信
api 25  MessageQueue.java 
// Called from native code.
private int dispatchEvents




Android7.0源码剖析 android Handler机制，从java到c++   epoll
https://www.jianshu.com/p/bffb1a9b985a 
调用java层的方法，就是MessageQueue.dispatchEvents:
https://segmentfault.com/a/1190000008685341
Android开发：详解Handler的内存泄露
https://blog.csdn.net/carson_ho/article/details/52693211

ThreadLocal在Android消息机制中的作用
https://www.jianshu.com/p/a8fa72e708d3

Android的消息机制之ThreadLocal的工作原理
https://blog.csdn.net/singwhatiwanna/article/details/48350919

[深入理解Android卷一全文-第五章]深入理解常见类
https://blog.csdn.net/innost/article/details/47207951

Android中消息系统模型和Handler Looper
https://www.cnblogs.com/bastard/archive/2012/06/08/2541944.html

