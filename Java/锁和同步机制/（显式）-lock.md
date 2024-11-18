# Lock
- 显式锁,需要手动开启和关闭
- 只有代码块锁
- 性能更好,JVM使用较少时间来调度线程
- 扩展性好(更多子类)

# ReentrantLock
可重入锁

```java
// 定义可重入锁
private final ReentrantLock lock = new ReentrantLock();

...

// 在方法中
// 加锁
lock.lock();

try{
    ...
}finally{
    // 解锁
    lock.unlock();
}
```