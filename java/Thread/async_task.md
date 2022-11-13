# 异步编程
将业务提交给Runnable实例task,如果用task.run()进行执行,则该方法会使当前线程阻塞,此时是等待同步;
当使用new Thread(task).start()时,创建一个新的线程去执行task,当前线程可以继续向下运行,这种方式为异步编程.
Java提供了Executor接口来处理任务Runnable or Callable, 使任务创建方和执行方解藕. 但是Executor只有一个方法 execute,  无法感知任务的执行过程(异常、报错、执行结果等),也无法释放空闲线程

### 因此, Java使用ExecutorService接口继承了Executor,定义了对任务状态的获取方法(shutDown, awaitTermination等方法)

### AbstractExecutorService作为Executor的实现类, 定义了方法的具体实现

### ThreadPoolExecutor是AbstractExecutorService的默认实现类, 能够接收Runnable和Callable方法,并返回一个Future对象,用来获取任务执行结果.
ThreadPoolExecutor中维护了一个阻塞队列 BlockingQueue<Runnable>用来接收任务. 不同阻塞队列适用于不同的场景:

### Executors工厂类,提供了创建ThreadPoolExecutor的几种方式
1、执行大量耗时短的任务: Executors.newCachedThreadPool(). 内部采用SynchronousQueue 
2、固定线程数量的: Executors.newFixedThreadPool() 内部采用LinkedBlockingQueue. 当该线程池不被使用时应手动关闭, 否则会占用线程资源
3、单一消费模式: Executors.newSingleThreadPool()
4、周期性执行任务: Executors.newScheduledThreadPool() 采用ThreadPoolExecutor实现类ScheduledThreadPoolExecutor, 其内部采用了DelayedWorkQueue 

### CompletionService 提供了从ExecutorService中获取任务结果的方法


