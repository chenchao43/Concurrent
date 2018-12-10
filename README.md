
线程的六种状态：New，Runnable，Blocked，Waiting，Timed waiting，Terminated
注意一个Runnable线程可能正在运行也可能没有运行，取决于操作系统的调度，调用start方法，进入Runnable状态
没有强制线程终止的方法，调用interrupt方法可以用来请求中断线程。当在一个阻塞的线程上调用interrupt是，会出现异常
### 同步
#### 锁对象
两种方法，第一synchronized，第二java.util.concurrent包中的ReentrantLock
```
//使用lock
mylock.lock();
try{
    critical section
}
finally
{
    mylock.unlock();
}
```
可重入锁意味着可以重复地获得已经持有的锁，会有一个holdcount来计数，多一个方法会+1，方法退出会-1，为0时线程会释放锁。
#### 条件对象
通常线程进入临界区，却发现再某一条件满足之后它才能执行，要使用一个条件对象来管理那些已经获得了一个锁但是却不能做有用工作的线程。
```
private Condition sufficientFunds;
sufficientFunds =bankLock.NewCondition();
//insufficient
sufficientFunds.await();

```
等待获得锁的线程和调用await方法的线程有本质的不同，调用await()意味着进入某个条件的等待集，当锁可用时，线程并不能马上解除阻塞，直到另一个线程调用同一条件上的signalAll方法时为止。比如
```
sufficientFunds.signalAll();
```
激活等待的线程后，线程从await调用返回，一旦获得锁，从被阻塞的地方继续执行，通常需要再测试阻塞条件，因为signalAll只是说明可能满足条件，因此，await一般这样使用
```
while(!(ok to proceed))
{
    condition.await();    
}
```
线程调用await后，寄希望于其他线程来解锁，因此要注意死锁问题，一般，在对象的状态有利于等待线程的方向改变时调用signalAll，signal方法是随机解除一个，所以更容易死锁。
#### synchronized关键字
java中的每一个对象都有一个内部锁，synchronized作用换句话说
```
public synchronized void method()
{
    method body
}
//等价于
public void method()
{
    this.intrinsiclock.lock();
    try
    {
        method body
    }
    finally
    {
       this.intrinsiclock.unlock(); 
    }
}
```
内部锁只有一个相关条件，wait对应await，notify和notifyAll对应signal和signalAll，用法一样。
静态方法使用synchronized会获得class的锁。有一些局限
1. 不能中断一个正在试图获得锁的线程
2. 不能设置超时
3. 只有一个条件，可能不够
#### 同步阻塞
类似于synchronized(obj)
#### volatile
假设对共享变量除了赋值之外并不完成其他操作，那么使用volatile可以不用使用锁。
### 阻塞队列
通过使用一个或多个队列。生产者线程向队列插入元素，消费者线程取出他们，向满的队列添加元素或者空的队列去除元素会导致线程阻塞，如add、remove、element会抛出异常，多线程环境中，队列会在任何时候空或满，因此一定要使用offer、poll、peek方法替代，这些方法会返回false而不会抛出异常。带超时的offer和poll方法表示在规定时间内完成则返回true。
```
LinkedBlockingQueue//没有上边界，但可以选择制定容量
ArrayBlockingQueue//要制定容量，可以设置公平参数
PriorityBlockingQueue//元素按优先级被移除，如果队列空，取元素会阻塞线程
DelayQueue//实现Delayed接口，只有时间用完才能被移出队列
```
### 执行器
如果程序中创建了大量生命周期很短的线程，应该使用线程池，因为线程的构造和销毁是有一定代价的。将Runnable对象交给线程池，就会有线程来调用run方法，run方法退出时，线程不会死亡，而是准备下次服务
### 同步器
管理相互合作的线程集的类，一般如果满足这些条件，应该直接使用这些类而不是自己创建锁与条件
信号量、CountDownLatch、CyclicBarrier、交换器、同步队列
