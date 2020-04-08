# jdk 源码阅读

>1，在读Spring源码前，一定要先看看《J2EE Design and Development》

## 相关链接
* [adopt openjdk](https://adoptopenjdk.gitbooks.io/adoptopenjdk-getting-started-kit/en/)
* [build openjdk](http://cr.openjdk.java.net/~ihse/demo-new-build-readme/common/doc/building.html)
* [jd-gui反汇编工具](https://github.com/java-decompiler/jd-gui/releases)

## 如何阅读jdk源码
>1，设有目的，带这问题去读
>2，多问为什么，看见不清楚的，就要搞清楚，不要假装清楚
>3，多比较(横向，纵向)，总结
>4，有重点的读
>5,带着问题阅读源码，忽略不必要的细节，死磕重要的细节

### 网友提出的几个问题举例：以ConcurrentHashMap为例
```
（1）ConcurrentHashMap与HashMap的数据结构是否一样？

（2）HashMap在多线程环境下何时会出现并发安全问题？

（3）ConcurrentHashMap是怎么解决并发安全问题的？

（4）ConcurrentHashMap使用了哪些锁？

（5）ConcurrentHashMap的扩容是怎么进行的？

（6）ConcurrentHashMap是否是强一致性的？

（7）ConcurrentHashMap不能解决哪些问题？

（8）ConcurrentHashMap除了并发安全，还有哪些与HashMap不同的地方，为什么要那么实现？

（9）ConcurrentHashMap中有哪些不常见的技术值得学习？
```

### 其他类如何隐性继承Object
>1，当一个类没有显式标明继承的父类时，虚拟机会为其指定一个默认的父类（一般为Object）,经过编译与反编译，可以排除不是编译器默认加上的，应该是虚拟机默认加上的

### 序列化是如何实现的，为什么implements java.io.Serializable

### 如何做一个maven依赖工具包啊，这样我们就不用总是重复造轮子了

### 字符和字符串编码与长度，GB2312,UTF-8,UTF-16

### volatile，原子性与并发
>1，volatile修饰的变量不允许线程内部缓存和重排序，即直接修改内存
>2，可见性，是指线程之间的可见性，一个线程修改的状态对另一个线程是可见的
>3，原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性

### 原子性：循环CAS和锁机制
>1，使用循环CAS实现原子操作：JVM中的CAS操作利用处理器提供的CMPXCHG指令实现。自旋CAS实现的基本思路就是循环进行CAS操作直到成功为止
>2，CAS仍然存在三大问题：1）ABA问题，2）循环时间长开销大，3）只能保证一个共享变量的原子操作
>3，使用锁机制实现原子操作（除了偏向锁，JVM实现锁的方式都用到的循环CAS）
>4，ABA问题：如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。ABA问题的解决思路就是使用版本号

* [原子操作原理-重要-讲到硬件和汇编指令](https://blog.csdn.net/a934270082/article/details/51133253)
>1，总线锁：LOCK#信号，使只有一个处理器占用贡献内存
>2，缓存锁：

```openjdk/hotspot/src/share/vm/prims/unsafe.cpp
大致的逻辑：一个旧值，一个新值，一个旧值的引用，先判断一下旧值和旧值应用指向的地方的值是否相等，如果相等，说明可以重新设置这个值为新值，这里就会有ABA的可能（版本号解决）。。。
问题1:如何保证这个逻辑步骤是原子的
答：根据当前处理器的类型来决定是否为cmpxchg指令添加lock前缀

UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapObject(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jobject e_h, jobject x_h))
  UnsafeWrapper("Unsafe_CompareAndSwapObject");
  oop x = JNIHandles::resolve(x_h);// 新值
  oop e = JNIHandles::resolve(e_h);// 预期值
  oop p = JNIHandles::resolve(obj);
  HeapWord* addr = (HeapWord *)index_oop_from_field_offset_long(p, offset);// 在内存中的具体位置
  oop res = oopDesc::atomic_compare_exchange_oop(x, addr, e, true);
  jboolean success  = (res == e);// 如果返回的res等于e，则判定满足compare条件（说明res应该为内存中的当前值），但实际上会有ABA的问题
  if (success)// success为true时，说明此时已经交换成功（调用的是最底层的cmpxchg指令）
    update_barrier_set((void*)addr, x);// 每次Reference类型数据写操作时，都会产生一个Write Barrier暂时中断操作，配合垃圾收集器
  return success;
UNSAFE_END

1，如何，或者为什么要通过CAS来实现原子性，怎么理解
2，什么地方发生了自旋CAS
3，当if success=false，怎么办，会发生什么
```
``` 自旋CAS举例：
openjdk/jdk/src/classes/java/util/concurrent/atomic/AtomicInteger.java
 public final int incrementAndGet() {
        return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
    }
openjdk/jdk/src/classes/sun/misc/unsafe.java
public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));//自旋CAS

        return var5;
    }
```