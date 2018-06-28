## c++ concurrency in action

### basic

- 初始化std::thread 对象 临时对象导致被编译器解析成函数声明
    - 解决办法 两层（）或统一初始化方法{}
- join等待线程结束后程序真正结束，detach程序已经退出有可能线程仍在进行中，就有可能有一些诡异的动作，比如访问已经析构的对象引用等
- join ，如果线程开始而产生异常没到join,就漏了。
  - 解决方案，join guard
- detach 后台线程，一个模型 函数自己内部调用线程，datach，线程方法是函数本身，通过条件判断是处理事务还是另起新的线程...
- thread构造函数参数是复制 类似std::bind
  - 如果函数对象或函数方法参数是引用，肯定会有问题
    - 解决办法 std::ref
  - 如果参数不能被复制可以移动（smart_ptr）解决方法 std::move
- 线程的所有权 值语义，可移动不可复制 std::move+join_guard - >scope_thread
- std::thread::hardware_currency std::thread::id 

### 共享数据

- 竞态条件 互斥与无锁编程
- 软件事务内存 software transactional memory
- 接口中的竞态条件 
  - 解决方案 传引用 
  - 解决方案 异常安全的拷贝构造和移动构造
  - 解决方案，返回指针 shared_ptr
- 锁的细粒度 以及死锁的解决方案
  - 避免嵌套锁
  - 持有锁，要保证接下来的动作没有危害（用户侧调用就很容易被坑，所以避免调用用户侧代码）
  - 固定顺序开锁解锁
    - 锁，分优先级，层次
    - unique_lock unlock...lock...
    - lock_guard
- 初始化时保护共享数据
  - std::once_flag std::call_once
  - meyer's singleton
- boost::shared_lock shared_lock<>



### 同步并发操作

- 等待事件与条件 

  - 忙等待
  - 条件变量
  - std::condition_variable notify_one wait
  - std::future  一次性时间
    - std::async 类似std::thread 不过最后阻塞的是future.get()
    - std::packaged_task<>
    - std::promise

- future FP

- 消息传递同步 wait().handle<>

  

### c++内存模型与原子操作

- 原子操作 store load read-modify-write
- 同步操作与顺序 
  - 内存顺序
  - 顺序一致与内存同步操作，代价。
  - 非顺序一致的memory-order 线程不闭合时间的顺序一致