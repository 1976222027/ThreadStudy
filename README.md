#线程学习 基础与进阶
说明：线程方面的只是
##内容概括

1android 常用四种调用线程的方法,及特殊线程HandlerThread;

2android线程间的通信

3android线程同步

##一Thread介绍：

###（1）Thread主要函数

run()//包含线程运行时所执行的代码 

start()//用于启动线程

sleep()/sleep(long millis)//线程休眠，交出CPU，让CPU去执行其他的任务，然后线程进入阻塞状态，sleep方法不会释放锁

yield()//使当前线程交出CPU，让CPU去执行其他的任务，但不会是线程进入阻塞状态，而是重置为就绪状态，yield方法不会释放锁

join()/join(long millis)/join(long millis,int nanoseconds)//等待线程终止，直白的说 就是发起该子线程的线程 只有等待该子线程运行结束才能继续往下运行

wait()//交出cpu，让CPU去执行其他的任务，让线程进入阻塞状态，同时也会释放锁

interrupt()//中断线程，自stop函数过时之后，我们通过interrupt方法和isInterrupted()方法来停止正在运行的线程，注意只能中断已经处于阻塞的线程

getId()//获取当前线程的ID

getName()/setName()//获取和设置线程的名字

getPriority()/setPriority()//获取和这是线程的优先级 一般property用1-10的整数表示，默认优先级是5，优先级最高是10，优先级高的线程被执行的机率高

setDaemon()/isDaemo()//设置和判断是否是守护线程

currentThread()//静态函数获取当前线程

###（2）Thread线程主要状态

（1） New  一旦被实例化之后就处于new状态

（2） Runnable 调用了start函数之后就处于Runnable状态

（3） Running 线程被cpu执行 调用run函数之后 就处于Running状态

 (4)   Blocked 调用join()、sleep()、wait()使线程处于Blocked状态

 (5)   Dead    线程的run()方法运行完毕或被中断或被异常退出，线程将会到达Dead状态
 
 ###（3）线程的两种启动方式
 开启线程后，子线程独自运行，不影响主线程的的运行，但是，当子线程运行完，需要将结果返回给UI线程，让UI线程更新ui，需要使用handler，这是android独有的方式
  handler的原生写法，需要再次封装成软饮用/弱引用：当子线程运行过程中，当前Act销毁，子线程结果返回给销毁的Act，会抛出异常使用弱引用后，销毁的act不会强制
  处理消息队列信息，避免异常 弱引用的handler,最好写在baseAct中，方便其他act使用。
  
  
线程的两种启动方式：
####（1）new Thread/自定义Thread

eg1：

        Thread thread0 = new Thread(new Runnable() {
            @Override
            public void run() {
                //耗时操作
              
            }
        });
        thread0.start();
        
 eg2:
 
         MyThread2 thread1 = new MyThread2("name1");
         thread1.start();  
         
              
 自定义Thread
  
         private class MyThread2 extends Thread {
     
             public MyThread2(String name) {
                 super(name);
             }
     
             @Override
             public void run() {
                 super.run();
         
             }
         } 
         
####（2）实现Runnable/自定义Runnable

  eg1：
    
        Thread thread1 = new Thread(this);
        thread1.start();
        
  而this，是指在Act类后，添加 implements Runnable,然后act类中重写 run方法：
  
      @Override
      public void run() {
          //
      }

eg2：

        Thread thread1 = new Thread(new MyRunnable());
        thread1.start();
        
这是自定义的Runnable 类

            private class MyRunnable implements Runnable {
        
                @Override
                public void run() {
                   
               }
            }
###（4）线程的构造解读：
总共有8个构造重载，05--08可以理解成一个重载，使用方式见代码，

####注：当自定义Thread(Runnable run1)类中还有重写的run方法，执行顺序是；先执行参数run1，再执行重写的run方法

        /**
         * 01 new 自定义无参thread,需在该线程中重写run方法。
         */
        public MyThread() {

        }

        /**
         * 02 给自定义的thread线程起一个名称
         *
         * @param name
         */
        public MyThread(String name) {
            super(name);
        }
        
        /**
         * 03 implement Runnable形式,该Runnable可以自定义
         *
         * @param target
         */
        public MyThread2(Runnable target) {
            super(target);
        }

        /**
         * 04 有名称的线程，使用Runnable形式
         */
        public MyThread2(Runnable target, String name) {
            super(target, name);
        }
        /**
         * 05
         *
         * @param group
         * @param target
         */
        public MyThread3(ThreadGroup group, Runnable target) {
            super(group, target);
        }

        /**
         * 06
         *
         * @param group
         * @param name
         */
        public MyThread3(ThreadGroup group, String name) {
            super(group, name);
        }

        /**
         * 07
         *
         * @param group
         * @param target
         * @param name
         */
        public MyThread3(ThreadGroup group, Runnable target, String name) {
            super(group, target, name);
        }

        /**
         * 08
         *
         * @param group     指定当前线程的线程组
         * @param target    需要指定，或者 在自定义线程中重写run
         * @param name      线程的名称，不指定自动生成
         * @param stackSize 预期堆栈大小，不指定默认为0
         */
        public MyThread3(ThreadGroup group, Runnable target, String name, long stackSize) {
            super(group, target, name, stackSize);
        }



 ##二HandlerThread介绍：
 HandlerThread特点:
 
 1本质是线程，继承Thread
 
 2HandlerThread内部有自己的Looper对象,可以在当前线程中处理分发消息
 
3通过获取HandlerThread的looper对象传递给Handler对象，可以在handleMessage方法中执行异步任务

优点：

1.开发中如果多次使用类似new Thread(){}.start()这种方式开启子线程，会创建多个匿名线程，使得程序运行起来越来越慢，
而HandlerThread自带Looper使他可以通过消息机制来多次重复使用当前线程，节省开支。
 
 2.Handler类内部的Looper默认绑定的是UI线程的消息队列，对于非UI线程如果需要使用消息机制，
自己去创建Looper较繁琐，由于HandlerThread内部已经自动创建了Looper，直接使用HandlerThread更方便


###常见用法：
用法1：Handler(Looper looper,Callback callback)执行异步

用法2：Handler(Looper looper）{handleMessage}执行异步

用法3：handler.post(runnable)执行异步

用法4:验证handler的post和重写handleMessage的执行顺序

##三 IntentService 异步介绍：
它本质是一种特殊的Service,继承自Service并且本身就是一个抽象类

它可以用于在后台执行耗时的异步任务，当任务完成后会自动停止

它拥有较高的优先级，不易被系统杀死（继承自Service的缘故），因此比较适合执行一些高优先级的异步任务

它内部通过HandlerThread和Handler实现异步操作

创建IntentService时，只需实现onHandleIntent和 空构造方法这两步骤即可，onHandleIntent为异步方法，可以执行耗时操作

###使用步骤：

 1:自定义的ItnentService重写onHandleIntent方法
 
 2：在Manifest.xml注册自定义的ItnentService
 
 3: startService(intent);启动服务
 
 
###常见用法：

方式1：自定义回调UI

方式2：LocalBroadcastManager触发广播回调UI

方式3：act触发广播回调UI

##四 线程池介绍：
###1Executors类详解：

 使用说明，其下有6类线程创建方式，还有其他三个辅助方法

  =========================================6类线程创建=======================================
  
  (1)
  
  public static ExecutorService newSingleThreadExecutor()
  
  public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory)
  
  用于创建只有一个线程的线程池
  hreadFactory，自定义创建线程时的行为

  (2)
  
  public static ExecutorService newCachedThreadPool()
  
  public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory)
  
  用于创建线程数数目可以随着实际情况自动调节的线程池
  当线程池有很多任务需要处理时，会不断地创建新线程，当任务处理完毕之后，如果某个线程空闲时间大于60s，则该线程将会被销毁。
  因为这种线程池能够自动调节线程数量，所以比较适合执行大量的短期的小任务
 
  (3)
  
  public static ScheduledExecutorService newSingleThreadScheduledExecutor()
  
  public static ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory threadFactory)
  
  用于创建只有一个线程的线程池，并且该线程定时周期性地执行给定的任务
  说明：线程在周期性地执行任务时如果遇到Exception，则以后将不再周期性地执行任务
 
  (4)
  
  public static ExecutorService newFixedThreadPool(int nThreads)
  
  public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory)
  
  用于创建一个最大线程数的固定的线程池，多余任务存储在队列中，等待线程池某一个线程完成任务，后续任务补充进去继续使用，总之，保持最大线程数不变
  threadFactory是线程工厂类，主要用于自定义线程池中创建新线程时的行为
  
  （5）
  
  public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
  
  public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory)
  
  用于创建一个线程池，线程池中得线程能够周期性地执行给定的任务

  (6)
  
  public static ExecutorService newWorkStealingPool(int parallelism)
  
  public static ExecutorService newWorkStealingPool()
  
  用于创建ForkJoin框架中用到的ForkJoinPool线程池，参数parallelism用于指定并行数，默认使用当前机器可用的CPU个数作为并行数。
  
  =============================================其他方法==============================================
  
  (7)
  
  public static ExecutorService unconfigurableExecutorService(ExecutorService executor)
  
  用于包装现有的线程池，包装之后的线程池不能修改，相当于final
  
  (8)
  
  public static ScheduledExecutorService unconfigurableScheduledExecutorService(ScheduledExecutorService executor)
  
  用于包装可以周期性执行任务的线程池，包装之后的线程池不能修改，相当于final
  
  (9)
  
  public static ThreadFactory defaultThreadFactory()
  
  返回默认的工厂方法类，默认的工厂方法为线程池中新创建的线程命名为：pool-[虚拟机中线程池编号]-thread-[线程编号
  
  (10)
  
  public static ThreadFactory privilegedThreadFactory()
  
  返回用于创建新线程的线程工厂，这些新线程与当前线程具有相同的权限。此工厂创建具有与 defaultThreadFactory() 相同设置的线程，
    新线程的 AccessControlContext 和 contextClassLoader 的其他设置与调用此 privilegedThreadFactory 方法的线程相同。可以在 AccessController.doPrivileged(java.security.PrivilegedAction) 操作中创建一个新 privilegedThreadFactory，设置当前线程的访问控制上下文，以便创建具有该操作中保持的所选权限的线程。
    注意，虽然运行在此类线程中的任务具有与当前线程相同的访问控制和类加载器，但是它们无需具有相同的 ThreadLocal
    或 InheritableThreadLocal 值。如有必要，使用 ThreadPoolExecutor.beforeExecute(java.lang.Thread, java.lang.Runnable)
    在 ThreadPoolExecutor 子类中运行任何任务前，可以设置或重置线程局部变量的特定值。
    另外，如果必须初始化 worker 线程，以具有与某些其他指定线程相同的 InheritableThreadLocal 设置，
    则可以在线程等待和服务创建请求的环境中创建自定义的 ThreadFactory，而不是继承其值。
    
 (11)
 
        public static Callable <Object> callable(Runnable task) 
  运行给定的任务并返回 null
   
        public static <T> Callable<T> callable(Runnable task,T result) 
  运行给定的任务并返回给定的结果。这在把需要 Callable 的方法应用到其他无结果的操作时很有用
   
        public static Callable<Object> callable(PrivilegedAction<?> action) 
  运行给定特权的操作并返回其结果