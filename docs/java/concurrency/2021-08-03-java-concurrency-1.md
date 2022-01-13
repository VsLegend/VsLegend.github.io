# Java并发 线程的异常捕获
----

## 通过Thread.UncaughtExceptionHandler捕获线程异常
Thread.UncaughtExceptionHandler类的作用：处理异常的接口，当线程由于未捕获的异常而突然终止时调用。

当线程由于未捕获的异常而即将终止时，Java虚拟机将使用getUncaughtExceptionHandler查询线程的UncaughtExceptionHandler并调用处理程序的uncaughtException方法，将线程和异常作为参数传递。 

如果一个线程没有显式设置它的UncaughtExceptionHandler ，那么它的ThreadGroup对象充当它的UncaughtExceptionHandler。 

如果ThreadGroup对象对异常没有进行特殊处理，那么它将调用转发给默认的未捕获异常处理程序。

- **演示**：
```java
Thread.UncaughtExceptionHandler exceptionHandler = (t, e) -> {
    System.out.println("报错线程:" + t.getName());
    System.out.println("线程抛出的异常:" + e);
};
Thread thread = new Thread(() -> {
    throw new RuntimeException("throw a new Exception ! ");
});
thread.setUncaughtExceptionHandler(exceptionHandler); // 设置异常处理器
thread.start();
```


## 继承ThreadPoolExecutor#afterExecute方法捕获线程池异常
该方法在执行给定Runnable调用时调用。 此方法也将由执行任务的线程调用。 
在ThreadPoolExecutor中，线程池在执行单个线程任务的步骤，如下所示：
```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // …………
            try {
                beforeExecute(wt, task); // task执行前的操作
                Throwable thrown = null;
                try {
                    task.run(); // 执行任务
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown); // task执行后的操作，也就包括了异常的捕获工作
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```

如果线程执行后，Throwable非null，则Throwable是导致线程执行突然终止的未捕获的RuntimeException或Error 。

这个实现在ThreadPoolExecutor中没有实现具体调用，需要子类继承然后自定规则。 注意：要正确嵌套多个覆盖，子类通常应在此方法的开头调用super.afterExecute，以确保不会破坏其他父类方法的实现。

>注意：当操作被显式或通过诸如submit等方法包含在任务（例如FutureTask ）中时，这些任务对象将自己捕获并维护计算中的异常，因此它们不会进入上面的catch分支，并且内部异常（Throwable将会为空）不会传递给该方法. 
>如果您想在此方法中捕获这两种故障，您可以进一步探测此类情况，如在此示例子类中，如果任务已中止，则打印直接原因或潜在异常，如下面的演示代码所示操作。


- **ThreadPoolExecutor中的演示代码**：
```java
class ExtendedExecutor extends ThreadPoolExecutor {
  protected void afterExecute(Runnable r, Throwable t) {
    super.afterExecute(r, t);
    // 如果任务是Future类型，那么异常将会直接通过Future返回
    if (t == null && r instanceof Future<?>) {
      try {
        Object result = ((Future<?>) r).get();
      } catch (CancellationException ce) {
          t = ce;
      } catch (ExecutionException ee) {
          t = ee.getCause();
      } catch (InterruptedException ie) {
          Thread.currentThread().interrupt(); // ignore/reset
      }
    }
    if (t != null)
      System.out.println(t);
  }
}
```