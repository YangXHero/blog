---
title: ThreadPoolTaskExecutor
date: 2019-03-21 10:43:02
tags:
    - Java SpringBoot
categories:
    - Java
---
{% blockquote %}
 ThreadPoolTaskExecutor是一个Spring的线程池技术，它是使用jdk中的java.util.concurrent.ThreadPoolExecutor进行实现。 
{% endblockquote %}

#### ThreadPoolTaskExecutor的参数：
    
    int corePoolSize:线程池维护线程的最小数量. 
    int maximumPoolSize:线程池维护线程的最大数量. 
    long keepAliveTime:空闲线程的存活时间. 
    TimeUnit unit: 时间单位,现有纳秒,微秒,毫秒,秒枚举值. 
    BlockingQueue<Runnable> workQueue:持有等待执行的任务队列. 
    RejectedExecutionHandler handler: 用来拒绝一个任务的执行，有两种情况会发生这种情况。 
        一是在execute方法中若addIfUnderMaximumPoolSize(command)为false，即线程池已经饱和； 
        二是在execute方法中, 发现runState!=RUNNING || poolSize == 0,即已经shutdown,就调用ensureQueuedTaskHandled(Runnable command)，在该方法中有可能调用reject。
        
#### ThredPoolTaskExcutor的处理流程：

    1.当池子大小小于corePoolSize，就新建线程，并处理请求
    2.当池子大小等于corePoolSize，把请求放入workQueue中，池子里的空闲线程就去workQueue中取任务并处理
    3.当workQueue放不下任务时，就新建线程入池，并处理请求，如果池子大小撑到了maximumPoolSize，就用RejectedExecutionHandler来做拒绝处理
    4.当池子的线程数大于corePoolSize时，多余的线程会等待keepAliveTime长时间，如果无请求可处理就自行销毁
    
    其会优先创建  CorePoolSiz 线程， 当继续增加线程时，先放入Queue中，当 CorePoolSiz  和 Queue 都满的时候，就增加创建新线程，当线程达到MaxPoolSize的时候，就会抛出错 误 org.springframework.core.task.TaskRejectedException
    另外MaxPoolSize的设定如果比系统支持的线程数还要大时，会抛出java.lang.OutOfMemoryError: unable to create new native thread 异常。
    
#### Reject策略预定义有四种： 
    (1)ThreadPoolExecutor.AbortPolicy策略，是默认的策略,处理程序遭到拒绝将抛出运行时 RejectedExecutionException。 
    (2)ThreadPoolExecutor.CallerRunsPolicy策略 ,调用者的线程会执行该任务,如果执行器已关闭,则丢弃. 
    (3)ThreadPoolExecutor.DiscardPolicy策略，不能执行的任务将被丢弃. 
    (4)ThreadPoolExecutor.DiscardOldestPolicy策略，如果执行程序尚未关闭，则位于工作队列头部的任务将被删除，然后重试执行程序（如果再次失败，则重复此过程）.
    
#### 使用。

##### SpringBoot  Config
###### 创建，初始化。
```Java
    /**
     *  默认线程池线程池
     * @author: yangkai
     * @date: 13:59 2019/03/28
     * @return: org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor
     **/
    @Bean
    public ThreadPoolTaskExecutor defaultThreadPool() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        //核心线程数目
        executor.setCorePoolSize(5);
        //指定最大线程数
        executor.setMaxPoolSize(15);
        //队列中最大的数目
        executor.setQueueCapacity(10);
        //线程名称前缀
        executor.setThreadNamePrefix("apiThreadPool_");
        //rejection-policy：当pool已经达到max size的时候，如何处理新任务
        //CALLER_RUNS：不在新线程中执行任务，而是由调用者所在的线程来执行
        //对拒绝task的处理策略
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        //线程空闲后的最大存活时间
        executor.setKeepAliveSeconds(60);
        //加载
        executor.initialize();
        return executor;
    }
```
###### 使用方式。
    
    第一种方式：@Async 注解。
        注： @Async所修饰的函数不要定义为static类型，这样异步调用不会生效
    
    第二种方式：
        1.IOC 注入Bean
            /** 通过注解引入配置 */
            @Resource(name = "defaultThreadPool")
            private ThreadPoolTaskExecutor executor;
        2.executor.execute(); 调用。