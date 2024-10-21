# ThreadLocal 是什么？
ThreadLocal 就是线程的本地变量，当你创建了一个ThreadLocal变量，那么访问这个变量的每个线程都会有这个变量的一个本地拷贝，多个线程操作这个变量的时候，实际是操作自己本地内存里面的变量，从而起到线程隔离的作用，避免了线程安全问题。

**不是说是为了线程安全而存在，而是为了线程隔离的需求，间接保证了线程安全。**

# ThreadLocal 有哪些使用场景？
1. **隔离用户信息<br>**
登陆鉴权，把用户信息存储在ThreadLocal中，用户信息每个线程存一份（会话管理中使用）。
2. **日志上下文存储、请求链路ID<br>**
   请求开始时生成一个唯一id，然后每次打印日志的时候都带上这个唯一id，查询日志时带上这个条件即可。<br>
   分布式链路追踪，需要存储本次请求的traceId。
3. **事务管理器，数据库连接，Session管理<br>**
Spring的事务管理器用的就是 ThreadLocal。<br>
4. **线程安全<br>**
比如日期工具类 SimpleDateFormat，它是线程不安全的，多线程情况下使用时，将它放入ThreadLocal中。
# ThreadLocal 它的底层是如何实现的？
每个线程（Thread，注意：不是 ThreadLocal ）都有一个 ThreadLocalMap，可以这么说 ThreadLocal 就是 ThreadLocalMap的一个工具类，比如 ThreadLocal 的 set 方法，我们看它的源码可以发现，
它先获取一个本地线程，通过本地线程拿到 ThreadLocalMap 对象，来进行 set 操作，ThreadLocalMap 它是一个 K-V 键值对的数据结构，key 就是 ThreadLocal，value 就是具体的值。
```java
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            map.set(this, value);
        } else {
            createMap(t, value);
        }
    }
```
```java
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
```
```java
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;
```
# 使用示例 —— 线程隔离
```java
public class ThreadLocalExample {  
  
    // 创建一个 ThreadLocal 变量  
    private static ThreadLocal<Integer> threadLocalValue = ThreadLocal.withInitial(() -> 0);

    static {
        threadLocalValue.set(3);
    }
    
    public static void main(String[] args) {  
  
        // 创建两个线程  
        Thread thread1 = new Thread(() -> {  
            // 在线程1中设置值  
            threadLocalValue.set(1);  
            System.out.println("Thread 1: " + threadLocalValue.get()); // 输出: Thread 1: 1  
        });  
  
        Thread thread2 = new Thread(() -> {  
            // 在线程2中设置值  
            threadLocalValue.set(2);  
            System.out.println("Thread 2: " + threadLocalValue.get()); // 输出: Thread 2: 2  
        });  
  
        // 启动线程  
        thread1.start();  
        thread2.start();  
  
        // 在主线程中访问值  
        System.out.println("Main thread: " + threadLocalValue.get()); // 输出: Main thread: 3  
    }  
}
```