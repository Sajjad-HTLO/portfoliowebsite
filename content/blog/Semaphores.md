---
title: "Thread safety with semaphores"
date: 2021-04-29T12:28:07+24:00
description: "Using semaphore to achieve thread-safety"
author: "Sajad Hayatlou"
type: "post"
---


### What is Multi-Threading

Muti-threading refers to a condition that multiple working threads running simultaneously in the application to improve performance and CPU utilization.
The other side of Multi-threading is its challenges and pitfalls that we should aware of them and take appropriate actions to achieve thread safety.

Java's memory model describes when one thread's actions are guaranteed to be visible to another.

{{< figure src="https://miro.medium.com/max/5120/1*h9HRzRZTkm1HGZFnC3fs5A.jpeg" style="float:left;" >}}


There are several common mechanisms to achieve thread-safety in a multi-threaded environment:

- Using thread-safe classes (immutable, properly locked classes, thread-safe collections, ...)

Or with the help of synchronization structures, like: 
- Synchronizers
- Locks
- Semaphores

In this post, I'll take a look at Semaphores and their best use cases.

### Semaphore

Here is the formal definition of a semaphore:

{{< quotate >}}
In computer science, a semaphore is a variable or abstract data type used to control access to a common resource by multiple processes and avoid critical section problems in a concurrent system such as a multitasking operating system

{{</ quotate >}}


The semaphore has two major types:
- Binary semaphore (mutexes)

A binary semaphore is restricted to values of zero or one, while a counting semaphore can assume any nonnegative integer value. A binary semaphore can be used to control access to a single resource. In particular, it can be used to enforce mutual exclusion for a critical section in the user code.

{{< figure src="http://www.embeddedlinux.org.cn/rtconforembsys/5107final/images/0602.jpg" style="float:left;" >}}

This ensures that only a single thread can access the critical section at most.


- Counting semaphore

With a counting semaphore, we can let a specific number of threads access the critical section at most, then, when the next thread wants to get into the critical section, it'll block until one of the currently running threads release the permit and leaves the critical section.

{{< figure src="http://www.embeddedlinux.org.cn/rtconforembsys/5107final/images/0603.jpg" style="float:left;" >}}


### Create a counting semaphore

Let's see how a simple `unbounded` counting semaphore looks like:

{{< highlight java >}}
public class CountingSemaphore {
	private int signals = 0;
	
	public synchronized void take() {
		signals++;
		notify();
	}
	
	public synchronized void release() throws InterruptedException {
		while(signals == 0) {
			wait();
		}
		signals--;
	}
}
{{< /highlight >}}

And, in case of needing a `bounded` counting semaphore, we can refactor the previous class by adding the `capacity`:

{{< highlight java >}}
public class BoundedSemaphore {
    int signal;
    int capacity;

    public BoundedSemaphore(int cap) {
        signal = 0;
        capacity = cap;
    }

    public synchronized void acquire() throws InterruptedException {
        while(signal == capacity) {
            wait();
        }

        if(signal == 0) {
            notifyAll();
        }

        signal ++;
    }

    public synchronized void release() throws InterruptedException {
        while(signal == 0) {
            wait();
        }

        if(signal == capacity) {
            notifyAll();
        }

        signal --;
    }
}
{{< /highlight >}}

You can see that if the signal reaches the capacity, calling to `acquire` will block the caller thread, also if the signal was zero, calling the `release` will result in blocking the client code too.


### Example of semaphore use case

Suppose we have 10 working threads trying to enter the critical section to execute a method, but we want to limit the concurrency to 5.
So, only 5 threads will be able to run concurrently at the time, here is the simulation:

{{< highlight java >}}
ExecutorService executor = Executors.newFixedThreadPool(10);

Semaphore semaphore = new Semaphore(5);

Runnable task = () -> {
    boolean permit = false;
    try {
        permit = semaphore.tryAcquire(1, TimeUnit.SECONDS);
        if (permit) {
            System.out.println("A permit acquired");
	      // Critical section, sleep for 5 seconds (execution time of the critical method)
            sleep(5);
        } else {
            System.out.println("Could not acquire a permit");
        }
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    } finally {
        if (permit) {
            semaphore.release();
        }
    }
}

IntStream.range(0, 10).forEach(i -> executor.submit(task));

stop(executor);

{{< /highlight >}}

And this is the output

{{< highlight html >}}
Semaphore acquired
Semaphore acquired
Semaphore acquired
Semaphore acquired
Semaphore acquired
Could not acquire semaphore
Could not acquire semaphore
Could not acquire semaphore
Could not acquire semaphore
Could not acquire semaphore
{{< /highlight >}}


### In Summary

The Semaphore is an excellent choice when you want to control the level of concurrency.

Also there is no spinning, hence no waste of resources due to no busy waiting. That is because threads intending to access the critical section are queued and could access the priority section when they are de-queued, which is done by the semaphore implementation itself, hence, unnecessary CPU time is not spent on checking if a condition is satisfied to allow the thread to access the critical section.



