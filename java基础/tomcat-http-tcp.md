# Tomcat

![tomcat框架结构](./tomcat-framework.png "tomcat框架结构")
![tomcat大致处理流程](./tomcat_struct.png "")

![tomcat大致处理流程](./tomcat-process.jpg "")

## 问题
>1，http如何基于TCP得到有效数据？
>2，tomcat源码如何学习和阅读，有什么方法

## TCP

### 半包&粘包
>1，半包：发送方发送的包太大，接收方没有接收完一个完整的包
>2，粘包：发送方和接收方都可能导致粘包。1）Nagle算法造成的发送端的粘包。2）接收方接收不及时
>3，解决办法：1，基于TLV封包，固定长度的包头（TL，type，length）和随机长度的数据Value，先提取包头，得到包长，得到有效包数据

### Nagle算法
>1，solve Small Packet Problem：有效数据部分一个字节，TCP头部20B，IP头部20B，为了1B，发送头部40B，导致拥塞
>2，**agle算法主要是避免发送小的数据包，要求TCP连接上最多只能有一个未被确认的小分组，在该分组的确认到达之前不能发送其他的小分组**
>3，linux提供了TCP_NODELAY的选项来禁用Nagle算法：setsockopt(client_fd, SOL_TCP, TCP_NODELAY,(int[]){1}, sizeof(int));
>4，禁止Nagle算法，每一次send，都会组一个包进行发送，不用等待ACK，可以连续发送

### Delay ACK and Nagle 导致死锁
>举一个场景：PC1和PC2进行通信，PC1发数据给PC2，PC1使用Nagle算法，PC2有delay ACK机制：
>1. PC1发送一个数据包给PC2，PC2会先不回应，delay ACK
>2. PC1再次调用send函数发送小于MSS的数据，这些数据会被保存到Buffer中，等待ACK，才能再次被发送
>从上面的描述看，显然已经死锁了，PC1在等待ACK，PC2在delay ACK，那么解锁的代价就是Delay ACK的Timer到期,40ms~500ms不等