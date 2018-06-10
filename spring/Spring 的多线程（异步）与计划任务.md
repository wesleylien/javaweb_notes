* [Spring 异步任务](#Spring 异步任务)
    * [Spring 异步任务的支持](#Spring 异步任务的支持)
        * [xml 配置方式](#xml 配置方式)
        * [Java 配置方式](#Java 配置方式)
    * [注解需要执行的异步任务的类或方法](#注解需要执行的异步任务的类或方法)
* [Spring 计划任务](#Spring 计划任务)
    * [Spring 计划任务的支持](#Spring 计划任务的支持)
    * [注解需要执行的计划任务的方法](#注解需要执行的计划任务的方法)

## Spring 异步任务
### Spring 异步任务的支持
#### xml 配置方式
    1. 配置基于线程池的 TaskExecutor
    2. 开启异步任务支持，指定默认的执行线程池的 TaskExecutor
    ```
    xmlns:task="http://www.springframework.org/schema/task"

    http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-4.1.xsd
    ```
    ```
    <!-- executor：指定一个缺省的executor给@Async使用 -->
    <task:annotation-driven executor="defaultExecutor"/>

    <!-- pool-size：线程池大小，格式为 core_size-max_size，core_size指最小的线程数，缺省为1；max_size指最大的线程数，缺省为Integer.MAX_VALUE -->
    <!-- queue-capacity：当最小的线程数已经被占用满后，新的任务会被放进queue里面，当这个queue的capacity也被占满之后，pool里面会创建新线程处理这个任务，直到总线程数达到了max_size，这时系统会拒绝这个任务并抛出TaskRejectedException异常（缺省配置的情况下，可以通过rejection-policy来决定如何处理这种情况）。缺省值为：Integer.MAX_VALUE -->
    <!-- keep-alive：超过core_size的那些线程，任务完成后，再经过这个时长（秒）会被结束掉 -->
    <!-- rejection-policy：当pool已经达到max size的时候，如何处理新任务
    ABORT（缺省）：抛出TaskRejectedException异常，然后不执行
    DISCARD：不执行，也不抛出异常
    DISCARD_OLDEST：丢弃queue中最旧的那个任务
    CALLER_RUNS：不在新线程中执行任务，而是有调用者所在的线程来执行
    -->
    <task:executor id="defaultExecutor" pool-size="100-10000" queue-capacity="10" keep-alive="5" rejection-policy="ABORT"/>
    ```
#### Java 配置方式
1. 配置类注解 `@EnableAsync` 开启异步任务支持
2. 实现 AsyncConfigurer 接口用于配置 TaskExecutor 和异常处理 Handler
    ```
    @Configuration
    @ComponentScan("com.lian")
    // 开启对异步任务的支持
    @EnableAsync  
    // 实现 AsyncConfigurer 接口并重写 getAsyncExecutor() 方法，以获得一个 TaskExecutor
    public class TaskExecutorConfig implements AsyncConfigurer {  

        @Override
        public Executor getAsyncExecutor() {
            // 使用 ThreadPoolTaskExecutor 可实现一个基于线程池的 TaskExecutor
            ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
            executor.setCorePoolSize(5);  
            executor.setMaxPoolSize(10);  
            executor.setQueueCapacity(25);
            executor.initialize();  
            return executor;
        }

        @Override
        public AsyncUncaughtExeceptionHandler getAsyncUncaughtExeceptionHandler() {
            return null;
        }
    }
    ```
    或者
    ```
    @Configuration  
    @EnableAsync  
    public class SpringConfig {  

        /** Set the ThreadPoolExecutor's core pool size. */  
        private int corePoolSize = 10;  
        /** Set the ThreadPoolExecutor's maximum pool size. */  
        private int maxPoolSize = 200;  
        /** Set the capacity for the ThreadPoolExecutor's BlockingQueue. */  
        private int queueCapacity = 10;  

        private String ThreadNamePrefix = "MyLogExecutor-";  

        @Bean  
        public Executor logExecutor() {  
            ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();  
            executor.setCorePoolSize(corePoolSize);  
            executor.setMaxPoolSize(maxPoolSize);  
            executor.setQueueCapacity(queueCapacity);  
            executor.setThreadNamePrefix(ThreadNamePrefix);  

            // rejection-policy：当pool已经达到max size的时候，如何处理新任务  
            // CALLER_RUNS：不在新线程中执行任务，而是有调用者所在的线程来执行  
            executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());  
            executor.initialize();  
            return executor;  
        }
    }
    ```

### 注解需要执行的异步任务的类或方法
* 通过 `@Async` 注解表明方法是异步方法
* `@Async` 可注解在类级别或方法级别
* `@Async` 如果注解在类级别，表示该类所有方法都是异步方法
* @Async("xxx") xxx 可指定 TaskExecutor，否则为缺省 TaskExecutor

```
@Component
public class AsyncTask {
    @Async
    public void executeAsyncTask() {
        ...
    }
}
```

## Spring 计划任务
### Spring 计划任务的支持
#### xml 配置方式
1. 配置基于线程池的 TaskScheduler
2. 开启计划任务支持，指定默认的执行线程池
```
xmlns:task="http://www.springframework.org/schema/task"

http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-4.1.xsd
```
```
<!-- scheduler：指定一个缺省的scheduler给 @Scheduled 使用 -->
<task:annotation-driven scheduler="taskScheduler" mode="proxy"/>

<task:scheduler id="qbScheduler" pool-size="10"/>
```

#### Java 配置方式   
通过 @EnableScheduling 开启计划任务支持
```
@Configuration
@ComponentScan("com.lian")
// 开启对计划任务的支持
@EnableScheduling
public class TaskScheduledConfig {  

}
```

### 注解需要执行的计划任务的方法
在 xml 配置文件中，可通过如下以配置文件方式配置需要执行的计划任务的方法，但一般使用注解的方式
```
<task:scheduled-tasks>
    <task:scheduled ref="taskJob" method="job1" cron="0 * * * * ?"/>
</task:scheduled-tasks>
```

* 通过 `@Scheduled` 注解表明方法是计划任务方法
* `@Scheduled` 有三个属性：fixedRate、fixedDelay、cron
* fixedRate 表示从上一个任务开始到下一个任务开始的间隔，单位是毫秒
* fixedDelay 表示从上一个任务完成开始到下一个任务开始的间隔，单位是毫秒
* cron 是 UNIX 系统下的定时任务

```
@Component
public class ScheduledTask {
    @Scheduled(cron="0 28 11 ? * *")
    public void executeScheduledTask() {
        ...
    }
}
```
