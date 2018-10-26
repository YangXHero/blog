---
title: Thread JDK1.5
date: 2018-10-24 11:26:35
tags:
    - Thread
categories:
    - Java
---
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0"
width=330 height=86 src="//music.163.com/outchain/player?type=2&id=25862794&auto=1&height=66"></iframe>

### 一、Java5 前时代
#### 1. 并发实现（两种）
> Java Green Thread (模拟线程实现方式。)

> Java Native Thread

#### 2. 局限性
    缺少线程管理的支持。（线程池）
    缺少“锁”API。
    缺少执行完成的状态。
    执行结果获取困难。
    Double Check locking 不确定性。

#### 3. Code 例子
```Java
    public class ThreadMain {
        public static void main(String[] args) {
            Thread thread = new Thread(new Runnable() {
                /**
                 * synchronized 关键字是是一种编程语言修饰符。
                 * */
                @Override
                public synchronized void run() {
                    System.out.printf("[Thread : %s] Hello , World... \n",Thread.currentThread().getName());
                }
            },"Sub");

            thread.start();
            System.out.printf("[Thread : %s] Starting......\n",Thread.currentThread().getName());
        }
    }
```

```Java
/**
 * 可完成的{@link Runnable}
 *
 * @author Yangkai
 * @create 2018-10-24-12:11
 */
public class CompletableRunnableMain {
    public static void main(String[] args) throws InterruptedException {
        CompletableRunnable completableRunnable = new CompletableRunnable();
        Thread thread = new Thread(completableRunnable, "Sub");

        thread.start();
        /**
         * Waits for this thread to die.
         * 等着线程执行结束。串行操作。
         **/
        thread.join();
        System.out.printf("[Thread : %s] Starting......\n", Thread.currentThread().getName());

        System.out.printf("running is completed .....: %s\n", completableRunnable.isCompleted());
    }

    private static class CompletableRunnable implements Runnable {
        private volatile boolean completed = false;

        @Override
        public void run() {
            System.out.printf("[Thread : %s] Hello , World... \n", Thread.currentThread().getName());
            completed = true;
        }

        public boolean isCompleted() {
            return completed;
        }
    }
}
```