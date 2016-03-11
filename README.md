# Quanta-training 研发部Android方向
#多线程与异步
###1.多线程概念
在过去单 CPU 时代，单任务在一个时间点只能执行单一程序。之后发展到多任务阶段，计算机能在同一时间点并行执行多任务或多进程。虽然并不是真正意义上的“同一时间点”，而是多个任务或进程共享一个 CPU，并交由操作系统来完成多任务间对 CPU 的运行切换，以使得每个任务都有机会获得一定的时间片运行。

随着多任务对软件开发者带来的新挑战，程序不在能假设独占所有的 CPU 时间、所有的内存和其他计算机资源。一个好的程序榜样是在其不再使用这些资源时对其进行释放，以使得其他程序能有机会使用这些资源。

再后来发展到多线程技术，使得在一个程序内部能拥有多个线程并行执行。一个线程的执行可以被认为是一个 CPU 在执行该程序。当一个程序运行在多线程下，就好像有多个 CPU 在同时执行该程序。

多线程比多任务更加有挑战。多线程是在同一个程序内部并行执行，因此会对相同的内存空间进行并发读写操作。这可能是在单线程程序中从来不会遇到的问题。其中的一些错误也未必会在单 CPU 机器上出现，因为两个线程从来不会得到真正的并行执行。然而，更现代的计算机伴随着多核 CPU 的出现，也就意味着不同的线程能被不同的 CPU 核得到真正意义的并行执行。

Java 是最先支持多线程开发的语言之一。

###2.线程与进程区别
一个进程包括由操作系统分配的内存空间，包含一个或多个线程。一个线程不能独立的存在，它必须是进程的一部分。一个进程一直运行，直到所有的非守候线程都结束运行后才能结束。

###3.线程的生命周期
**新建状态**：
使用 new 关键字和 Thread 类或其子类建立一个线程对象后，该线程对象就处于新建状态。它保持这个状态直到程序 start() 这个线程。   
**就绪状态**:
当线程对象调用了start()方法之后，该线程就进入就绪状态。就绪状态的线程处于就绪队列中，要等待JVM里线程调度器的调度。   
**运行状态**:
如果就绪状态的线程获取 CPU 资源，就可以执行run()，此时线程便处于运行状态。处于运行状态的线程最为复杂，它可以变为阻塞状态、就绪状态和死亡状态。   
**阻塞状态**:
如果一个线程执行了sleep（睡眠）、suspend（挂起）等方法，失去所占用资源之后，该线程就从运行状态进入阻塞状态。在睡眠时间已到或获得设备资源后可以重新进入就绪状态。   
**死亡状态**:
一个运行状态的线程完成任务或者其他终止条件发生时，该线程就切换到终止状态。   

###4.如何创建并运行java线程
Java提供了两种创建线程方法：
 - 继承Thread类本身
 - 实现Runable接口

1).继承Thread类本身  
 创建 Thread 子类的一个实例并重写 run 方法，run 方法会在调用 start()方法之后被执行。
```java
public class MyThread extends Thread {
   public void run(){
     System.out.println("MyThread running");
   }
}
```
 创建并运行 Thread 子类
```java
MyThread myThread = new MyThread();
myThread.start();
```
2).实现 Runnable 接口  
 新建一个实现了 java.lang.Runnable 接口的类
```java
public class MyRunnable implements Runnable {
   public void run(){
    System.out.println("MyRunnable running");
   }
}
```
 在 Thread 类的构造函数中传入 MyRunnable 的实例对象
```java
Thread thread = new Thread(new MyRunnable());
thread.start();
```
###5.创建线程哪种方法好？
实现 Runnable 接口
1.可以避免Java的单继承的特性带来的局限性  
2.适合多个具有相同代码的线程去处理同一资源的情况，把线程、代码和数据资源三者有效分离，较好地体现了面向对象的设计思想。

#异步任务
###1.同步与异步的区别？
同步就是指一个进程在执行某个请求的时候，若该请求需要一段时间才能返回信息，那么这个进程将会一直等待下去，直到收到返回信息才继续执行下去。  
异步是指进程不需要一直等下去，而是继续执行下面的操作，不管其他进程的状态。当有消息返回时系统会通知进程进行处理，这样可以提高执行的效率。  
同步问题多发生在多线程环境中的数据共享问题。即当多个线程需要访问同一个资源时，它们需要以某种顺序来确保该资源在某一特定时刻只能被一个线程所访问，如果使用异步，程序的运行结果将不可预料。因此，在这种情况下，就必须对数据进行同步，即限制只能有一个进程访问资源，其他线程必须等待。
###2.安卓为什么要引入异步任务？
Android程序刚启动时，会同时启动一个对应的主线程(MainThread)，这个主线程主要负责处理与UI相关的事件，即为UI线程。Android UI操作并不是线程安全的，简单来说就是不能从非UI线程来操纵UI组件，必须把所有的UI操作放在UI线程里。

假如我们在非UI线程中，比如在主线程中new Thread()另外开辟一个线程，然后直接在里面修改UI控件的值；此时会抛出下述异常： android.view.ViewRoot$CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views；   如果我们把耗时的操作都放在UI线程中的话，如果UI线程超过5s没有响应用于请求，那么 这个时候会引发ANR(Application Not Responding)异常，就是应用无响应；  
最后一点就是：Android 4.0后禁止在UI线程中执行网络操作，不然会报: android.os.NetworkOnMainThreadException。   

所以，Android的单线程模型有两条原则：  
1.不要阻塞UI线程
2.不要在UI线程之外访问Android UI组件  

###3.Android异步处理
 - 使用handler实现非UI线程更新UI界面
 - 使用AsyncTask异步更新UI界面
1).利用handler可以实现线程间的通信，我们可以在非UI线程发送消息到UI线程，最终让Ui线程来进行ui的操作。
![]()
UI线程中创建handler，重写handleMessage(Message msg)方法
```java
private Handler mHandler = new Handler() {  
        public void handleMessage (Message msg) {//处理消息  
            switch(msg.what) {  
              case "0":
                 texyview.setText((String) msg.obj);
                    break;
        }  
    };  
```
非UI线程中发送Message到消息队列
```java
        Message msg = Message.obtain();
        msg.obj = "data";
        msg.what = 0;
        mHandler.sendMessage(msg); //向Handler发送消息,更新UI
```

具体流程：新建一个Handler对象,通过这个对象向主线程发送信息;而我们发送的信息会先到主线程的MessageQueue进行等待,由Looper按先入先出顺序取出,再根据message对象的what属性分发给对应的Handler进行处理     
[Android异步消息处理机制完全解析，带你从源码的角度彻底理解](http://blog.csdn.net/guolin_blog/article/details/9991569)   
[android的消息处理机制（图+源码分析）——Looper,Handler,Message](http://www.cnblogs.com/codingmyworld/archive/2011/09/12/2174255.html)
2).AsyncTask
待续...
