
记录一次在@Test注解 环境下，ThreadPoolTaskExecutor多线程异常中断问题
<p style="color: red">[Thread-2] WARN org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor[awaitTerminationIfNecessary:246] - Timed out while waiting for executor 'taskExecutor' to terminate</p>

<br><br>

---

**测试代码如下**
```java
@ApplicationScope
@RunWith(SpringRunner.class)
@SpringBootTest
public class TemplateParserTest {
    
    @Resource
    private ThreadPoolTaskExecutor taskExecutor;

    @Test
    public void testSPackageTask() {
        AsyncThread task = new AsyncThread(); // extends Thread
        taskExecutor.execute(task); // 放进线程池执行
    }

}
```

该段代码主要是对新编写的线程方法进行测试，这一段方法主要是用于数据迁移，执行内部方法的时间比较长，此为起点，异常抛错如下。


**线程异常信息**

```
2022-04-02 14:26:46.624 [main] INFO TemplateParserTest[logStarted:59] - Started TemplateParserTest in 21.686 seconds (JVM running for 23.355)
[AsyncThread-1] INFO ServicePackageTemplateConvertTask[execServicePackageConvert:49] - SERVICE PACKAGE CONVERT----执行服务包转换，开始时间：2022-04-02T11:39:59.890+0800
[Thread-2] INFO org.springframework.web.context.support.GenericWebApplicationContext[doClose:993] - Closing org.springframework.web.context.support.GenericWebApplicationContext@4f63e3c7: startup date [Sat Apr 02 11:39:39 CST 2022]; root of context hierarchy
[Thread-2] INFO org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor[shutdown:208] - Shutting down ExecutorService 'taskExecutor'
[AsyncThread-1] INFO ServicePackageTemplateConvertTask[execServicePackageConvert:55] - SERVICE PACKAGE CONVERT----服务包总数：34852， 分段数：18, 生成模板起始ID：10000
[AsyncThread-1] INFO ServicePackageTemplateConvertTask[execServicePackageConvert:67] - SERVICE PACKAGE CONVERT----当前模板分段：0，转换结束, 成功转换条数2000
[AsyncThread-1] INFO ServicePackageTemplateConvertTask[execServicePackageConvert:67] - SERVICE PACKAGE CONVERT----当前模板分段：1，转换结束, 成功转换条数2000
[Thread-2] WARN org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor[awaitTerminationIfNecessary:246] - Timed out while waiting for executor 'taskExecutor' to terminate
```


<br><br>

---


**异常分析**

可以看到打印的线程信息，main线程执行了测试方法而后将任务提交至线程池，AsyncThread-1正是我们执行的线程任务，
而在第二三行，线程Thread-2正在关闭当前的@Test环境，并随即调用了taskExecutor线程池的shutdown方法，对线程池进行注销。

最后在执行一段时间的AsyncThread-1任务后（这时任务还没完成），Thread-2成功的将线程池关闭，此时执行的awaitTerminationIfNecessary方法。然后任务就中断了。

这里就出现了一个问题，为什么我们的AsyncThread-1任务还没执行结束，Thread-2就能成功的关闭线程池呢？

Spring中的ThreadPoolTaskExecutor类仅仅只是包装的ThreadPoolExecutor类，而在JUC包中ThreadPoolExecutor类，他的shutdown和shutdownNow都不会影响当前执行中的线程，
前者拒绝新任务提交，后者拒绝同时将不会执行队列中的任务。但是他们都会等待当前线程执行结束后，才会关闭线程池。


找不到头绪直接看源码


<br><br>

---


