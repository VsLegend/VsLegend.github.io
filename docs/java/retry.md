


- break retry：跳出循环，跳到retry处，且不再进入循环
- continue retry：跳出循环，跳到retry处，将再次进入循环

实例代码如ThreadPoolExecutor的addWorker方法所示

```java
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        // …………
        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loo
        }
    }
    boolean workerStarted = false;
    // …………
    return workerStarted;
}
```