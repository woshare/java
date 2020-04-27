# Java 面试问题

### final, finally, finalize的区别
>1，final—修饰符（关键字）如果一个类被声明为final，意味着它 **不能再派生出新的子类**，不能作为父类被继承。因此一个类不能既被声明为 abstract的，又被声明为final的。**将变量或方法声明为final，可以保证它们在使用中不被改变。** 被声明为final的变量必须在声明时给定初值，而在以后的引用中只能读取，不可修改。被声明为final的方法也同样只能使用，不能重载
>2，finally—再异常处理时提供 finally 块来执行任何清除操作。如果抛出一个异常，那么相匹配的 catch 子句就会执行，然后控制就会进入 finally 块（如果有的话）
>3，finalize—方法名。Java 技术允许使用 finalize()方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。**这个方法是由垃圾收集器在确定这个对象没有被引用时对这个对象调用的。** 它是在 Object 类中定义的，因此所有的类都继承了它。子类覆盖 finalize() 方法以整理系统资源或者执行其他清理工作。finalize() 方法是在垃圾收集器删除对象之前对这个对象调用的

### is-a & is-like-a
>1，is-a：继承只覆盖基类的方法，子类父类相同
>2，is-like-a：继承覆盖了基类的方法，且拓展新的成员元素

### 面向对象编程三特性
>1，封装,继承,多态
>2，继承是子类使用父类的方法，而多态是父类使用子类的方法
>3，如何实现多态，其对象内存模型什么样的？
>4，java单继承于Object，c++多继承

### 覆盖（override），重载（overload），重写（overwrite）
![Alt text](./override-overload-overwrite.png "覆盖（override），重载（overload），重写（overwrite）")

### finalize()
>1，对象可能不被垃圾回收
>2，垃圾回收不等同于析构
>3，创建的对象不是通过 new 来分配内存的，而垃圾回收器只知道如何释放用 new 创建的对象的内存，所以它不知道如何回收不是 new 分配的内存
>4，当垃圾回收器准备回收对象的内存时，首先会调用其 finalize() 方法，并在下一轮的垃圾回收动作发生时，才会真正回收对象占用的内存

### String str=new String("hello")和String str="hello"的区别
>1，String str1 = "Hello"; String str2 = "Hello";String str3 = "Hello";3个变量统一指向同一个堆内存地址，"Hello"放在堆内存常量池
![Alt text](./java-string=.png "String赋值方式一：常量池")
![Alt text](./java-string=&new-different.png "String赋值方式二：new")

###  extends 继承类；implements 实现接口

### =赋值与clone，赋值，浅克隆和深度克隆
```
浅克隆：
 @Override  
    public Object clone() {              // 下面 这个是重点
        Address addr = null;  
        try{  
            addr = (Address)super.clone();  
        }catch(CloneNotSupportedException e) {  
            e.printStackTrace();  
        }  
        return addr;  
    } 
```
```
深度克隆：如果引用类型里面还包含很多引用类型，或者内层引用类型的类里面又包含引用类型，使用clone方法就会很麻烦。这时我们可以用序列化的方式来实现对象的深克隆

public class Outer implements Serializable{
  private static final long serialVersionUID = 369285298572941L;  //最好是显式声明ID
  public Inner inner;
　//Discription:[深度复制方法,需要对象及对象所有的对象属性都实现序列化]　
  public Outer myclone() {
      Outer outer = null;
      try { // 将该对象序列化成流,因为写在流里的是对象的一个拷贝，而原对象仍然存在于JVM里面。所以利用这个特性可以实现对象的深拷贝
          ByteArrayOutputStream baos = new ByteArrayOutputStream();
          ObjectOutputStream oos = new ObjectOutputStream(baos);
          oos.writeObject(this);
　　　　　　// 将流序列化成对象
          ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
          ObjectInputStream ois = new ObjectInputStream(bais);
          outer = (Outer) ois.readObject();
      } catch (IOException e) {
          e.printStackTrace();
      } catch (ClassNotFoundException e) {
          e.printStackTrace();
      }
      return outer;
  }
}
```
>实现对象克隆有两种方式：
>1). 实现Cloneable接口并重写Object类中的clone()方法；
>2). 实现Serializable接口，通过对象的序列化和反序列化实现克隆，可以实现真正的深度克隆

### https
>1，CA ROOT 给server生成证书（CA的私钥加密server的信息（1，公钥：Public-Key 2，签名：Signature
3,签名算法： Signature Algorithm: sha256WithRSAEncryption4，证书颁布机构：Issuer 5，过期时间：Validity...））
>2，client 有CA ROOT的证书公钥
>3，client发起https请求
>4，server回应client证书
>5，client用CA的公钥解开server证书，拿到server的公钥
>6，client生成随机对称秘钥，用server的公钥加密，给server
>7，server拿到client的加密数据，用自己的私钥解开，拿到client的对称秘钥
>8，用对称秘钥加解密传输数据

### hashcode，equals和toString，都需要重写
>1，自反性，传递性，一致性等
>2，成对重写，写了hashcode就要写equals

### 构造器注意编写，因为构造器在初始化中途出现异常，容易导致内存没法释放

### 线程池的使用
* [线程池源码介绍和使用](https://www.cnblogs.com/dafanjoy/p/9729358.html)
* [线程池源码解读](https://mp.weixin.qq.com/s/4LP-kIVUGPe8H8nBB6-HQA)
* [线程池源码解读-有图示，更详细](https://www.cnblogs.com/KingJack/p/9595621.html)
#### 线程数的设定
``` 
            /**
             * Nthreads=CPU数量
             * Ucpu=目标CPU的使用率，0<=Ucpu<=1
             * W/C=任务等待时间与任务计算时间的比率
             */
            Nthreads = Ncpu*Ucpu*(1+W/C)
```
#### ThreadPoolExecutor扩展
>1、beforeExecute：线程池中任务运行前执行
>2、afterExecute：线程池中任务运行完毕后执行
>3、terminated：线程池退出后执行
>4，在Worker类中，run()->runWorker()->{beforeExecute();task.run();afterExecute();},在ThreadPoolExecutor类中，这三个函数式空函数，可以拓展，以实现工作线程前后和线程终结时机的操作
>5，execute(Runnable command)->addWorker(command, true)->{workers.add(w);final Thread t = w.thread;t.start();//这个地方执行了线程，worker->run()->runWorker()->{beforeExecute();task.run();afterExecute();}}
>6,shutdownNow()会中断所有的存活线程，不论这些线程是否空闲，因此可能会导致任务在执行的过程中抛出异常，这点需要注意
>7,shutdown()方法只会中断空闲线程，但是非空闲的线程不会被中断，即使该线程被阻塞，因此该方法有可能无法关闭那些一直处在等待状态的非空闲线程，这一点在使用时需要注意
>8,threadFactory:线程工厂, 如果没有设定线程工厂，那会使用DefaultThreadFactory，在Executor.java中有实现ThreadFactory接口的 newThread 
>9，**线程异常，会使得不能捕获从线程中逃逸（抛出）的异常，要额外使用Thread.UncaughtExceptionHandler.uncaughtException** ，不过还不太懂

#### 几种常用的线程池
>1，newScheduledThreadPool：创建一个定长的线程池，而且支持定时的以及周期性的任务执行，支持定时及周期性任务执行 
>2，newSingleThreadExecutor：只创建唯一的工作者线程来执行任务，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序
>3，newFixedThreadPool：指定工作线程数量的线程池。每当提交一个任务就创建一个工作线程，如果工作线程数量达到线程池初始的最大数，则将提交的任务存入到池队列中。
>4，newCachedThreadPool：创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程

#### 线程，怎么就是线程了，它和别的模块有什么不一样呢，就实现了线程的功能了呢，什么是线程并是如何实现线程的

#### 线程有返回值，可以实现Callable接口，Runnable接口没有返回值

#### 线程异常捕获，什么线程特性（run 函数没有throw和catch？），会使得不能捕获从线程中逃逸（抛出）的异常，要额外使用Thread.UncaughtExceptionHandler.uncaughtException

>在java多线程程序中，所有线程都不允许抛出未捕获的checked exception（比如sleep时的InterruptedException），也就是说各个线程需要自己把自己的checked exception处理掉。这一点是通过java.lang.Runnable.run()方法声明(因为此方法声明上没有throw exception部分)进行了约束。但是线程依然有可能抛出unchecked exception（如运行时异常），当此类异常跑抛出时，线程就会终结，而对于主线程和其他线程完全不受影响，且完全感知不到某个线程抛出的异常(也是说完全无法catch到这个异常)。JVM的这种设计源自于这样一种理念：“线程是独立执行的代码片断，线程的问题应该由线程自己来解决，而不要委托到外部。”基于这样的设计理念，在Java中，线程方法的异常（无论是checked还是unchecked exception），都应该在线程代码边界之内（run方法内）进行try catch并处理掉.换句话说，我们不能捕获从线程中逃逸的异常
>1，线程是独立执行的代码片断，线程的问题应该由线程自己来解决，而不要委托到外部
>2，线程不能被外部捕获，要么就自定义uncaughtException在线程内部捕获处理异常，但是uncaughtException又是如何捕获线程异常的呢
>3，class ThreadGroup implements Thread.UncaughtExceptionHandler：ThreadGroup类中实现了uncaughtException

>根据JDK文档的描述，当出现未捕获异常的时候，JVM调用的是Thread.getUncaughtExceptionHandler()，这个方法中，如果你没有调用Thread.setUncaughtExceptionHandler()设置未捕获异常处理器，那么将会返回Thread.group，将ThreadGroup当作未捕获异常处理器，而ThreadGroup实现了UncaughtExceptionHandler，所以转到ThreadGroup的uncaughtException(Thread, Throwable)方法

* (讲清楚了线程未捕获异常Thread.UncaughtExceptionHandler的源码)[https://yuanfentiank789.github.io/2017/05/11/uncaughtexception/]

#### @CallerSensitive 和反射调用的权限相关，但是不懂，说是为了堵JVM的漏洞用的

### synchronized与lock的区别，使用场景。看过synchronized的源码没
>1，Syncronized 的目的是一次只允许一个线程进入由他修饰的代码段，从而允许他们进行自我保护
>2，Syncronized 是关键字，Lock是一个接口
>3，1）获取锁的线程执行完同步代码后，自动释放；2. 线程发生异常时，JVM会让线程释放锁；Lock 必须在 finally 关键字中释放锁，不然容易造成线程死锁
>4，Synchronized 是可重入，不可中断，非公平锁；Lock 锁则是 可重入，可判断，可公平锁
>5，Synchronized 适用于少量同步的情况下，性能开销比较大。Lock 锁适用于大量同步阶段

### synchronized代码块同步是使用monitorenter 和monitorexit指令实现的，而方法同步是使用另外一种方式实现的。monitorenter指令是在编译后插入到同步代码块的开始位置，而monitorexit是插入到方法结 束处和异常处，JVM要保证每个monitorenter必须有对应的monitorexit与之配对

>1,synchronized用的锁是存在Java对象头里的。如果对象是数组类型，则虚拟机用3个字宽 （Word）存储对象头，如果对象是非数组类型，则用2字宽存储对象头。在32位虚拟机中，1字宽 等于4字节，即32bit

### 避免死锁的几个常见方法。
>1，·避免一个线程同时获取多个锁。
>2，·避免一个线程在锁内同时占用多个资源，尽量保证每个锁只占用一个资源。
>3，·尝试使用定时锁，使用lock.tryLock（timeout）来替代使用内部锁机制。
>4，·对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败的情况




### JVM自动内存管理，Minor GC与Full GC的触发机制


### Java锁有哪些种类，以及区别

### 主要就是并发编程中锁和锁的实现，原子操作，线程池，线程，jvm内存管理和调优

#### 锁
>偏向锁指的是当前只有这个线程获得，没有发生争抢，此时将方法头的markword设置成0，然后每次过来都cas一下就好，不用重复的获取锁
>轻量级锁：在偏向锁的基础上，有线程来争抢，此时膨胀为轻量级锁，多个线程获取锁时用cas自旋获取，而不是阻塞状态
>重量级锁：轻量级锁自旋一定次数后，膨胀为重量级锁，其他线程阻塞，当获取锁线程释放锁后唤醒其他线程。（线程阻塞和唤醒比上下文切换的时间影响大的多，涉及到用户态和内核态的切换）
>自旋锁：在没有获取锁的时候，不挂起而是不断轮询锁的状态
>公平锁/非公平锁
>可重入锁
>独享锁/共享锁
>互斥锁/读写锁
>乐观锁/悲观锁
>分段锁
>偏向锁/轻量级锁/重量级锁

#### 如何保证内存可见性
>1,volatile 通过内存屏障
>2,synchronized 通过修饰的程序段同一时间只能由同一线程运行，释放锁前会刷新到主内存

### Spring源码没，说说Ioc容器的加载过程

### 代理的实现原理

### BIO & NIO & AIO
>1，Java BIO ： 同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。
>2，Java NIO ： 同步非阻塞，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理
>3，Java AIO(NIO.2) ： 异步非阻塞，服务器实现模式为一个有效请求一个线程，客户端的I/O请求都是由OS先完成了再通知服务器应用去启动线程进行处理

>4，在高性能的I/O设计中，有两个比较著名的模式Reactor和Proactor模式，其中Reactor模式用于同步I/O，而Proactor运用于异步I/O操作
``` BIO：
 ServerSocket serverSocket;
        try {
            serverSocket = new ServerSocket(8000);
            while (true){
                Socket socket = serverSocket.accept();
                new Thread(()->{
                    try (InputStream inputStream = socket.getInputStream(); OutputStream outputStream =  socket.getOutputStream()) {

                        byte[] bytes =new byte[1024];
                        while (inputStream.read(bytes) != -1){
                            outputStream.write(bytes);
                            bytes = new byte[1024];
                        }

                    }catch (IOException e){
                        e.printStackTrace();
                    }
                }).start();

            }
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
```
``` NIO
Selector selector = Selector.open();  
ServerSocketChannel ssc = ServerSocketChannel.open();  
ssc.configureBlocking(false);  
ssc.socket().bind(new InetSocketAddress(port));  
  
ssc.register(selector, SelectionKey.OP_ACCEPT);  
  
while (true) {  
  
    // select()阻塞，等待有事件发生唤醒  
    int selected = selector.select();  
  
    if (selected > 0) {  
        Iterator<SelectionKey> selectedKeys = selector.selectedKeys().iterator();  
        while (selectedKeys.hasNext()) {  
            SelectionKey key = selectedKeys.next();  
            if ((key.readyOps() & SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT) {  
              SocketChannel client = ((ServerSocketChannel) key.channel()).accept();
                // 处理 accept 事件  
                //注册read事件
                client.configureBlocking(false);
                client.register(key.selector(), SelectionKey.OP_READ, ByteBuffer.allocate(bufSize));

            } else if ((key.readyOps() & SelectionKey.OP_READ) == SelectionKey.OP_READ) {  
                // 处理 read 事件  
               //注册write事件
            } else if ((key.readyOps() & SelectionKey.OP_WRITE) == SelectionKey.OP_WRITE) {  
                // 处理 write 事件  
            }  
            selectedKeys.remove();  
        }  
    }  
} 
```