#[进程（process） v.s. 线程(Thread) v.s. 协程(Coroutine)](https://zhuanlan.zhihu.com/p/114453309)

## 进程

进程是资源（CPU、内存等）分配的基本单位，它是程序执行时的一个实例。程序运行时系统就会创建一个进程，并为它分配资源，然后把该进程放入进程就绪队列，进程调度器选中它的时候就会为它分配CPU时间，程序开始真正运行。

Linux系统函数`fork()` 可以在父进程中创建一个子进程，这样的话，在一个进程接到来自客户端新的请求时就可以复制出一个子进程让其来处理，父进程只需负责监控请求的到来，然后创建子进程让其去处理，这样就能做到并发处理。

```python
import os

print('当前进程:%s 启动中 ....' % os.getpid())
pid = os.fork()
if pid == 0:
    print('子进程:%s,父进程是:%s' % (os.getpid(), os.getppid()))
else:
    print('进程:%s 创建了子进程:%s' % (os.getpid(),pid ))
```

输出结果：

```python
当前进程:27223 启动中 ....
进程:27223 创建了子进程:27224
子进程:27224,父进程是:27223
```

fork函数会返回两次结果，因为操作系统会把当前进程的数据复制一遍，然后程序就分两个进程继续运行后面的代码，fork分别在父进程和子进程中返回，在子进程返回的值pid永远是0，在父进程返回的是子进程的进程id。

## 线程

线程是程序执行时的最小单位，它是进程的一个执行流，是CPU调度和分派的基本单位，一个进程可以由很多个线程组成，线程间共享进程的所有资源，每个线程有自己的堆栈和局部变量。线程由CPU独立调度执行，在多CPU环境下就允许多个线程同时运行。同样多线程也可以实现并发操作，每个请求分配一个线程来处理。

## 协程

是一种比线程更加轻量级的存在。一个线程也可以拥有多个协程。其执行过程更类似于子例程，或者说不带返回值的函数调用。



## 进程和线程区别

### 地址空间

线程共享本进程的地址空间，而进程之间是独立的地址空间。 

###资源

线程共享本进程的资源如内存、I/O、cpu等，不利于资源的管理和保护，而进程之间的资源是独立的，能很好的进行资源管理和保护。 

###Robustness

多进程要比多线程健壮，一个进程崩溃后，在保护模式下不会对其他进程产生影响，但是一个线程崩溃整个进程都死掉。

### 执行过程

每个独立的进程有一个程序运行的入口、顺序执行序列和程序入口，执行开销大。

但是线程不能独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制，执行开销小。

### 均可并发

### 切换时（Context Switching?）

进程切换时，消耗的资源(memory addresses, page tables, and kernel resources, caches in the processor)大，效率高。所以涉及到频繁的切换时，使用线程要好于进程。同样如果要求同时进行并且又要共享某些变量的并发操作，只能用线程不能用进程

###其他

线程是处理器调度的基本单位，但是进程不是。

## Pros & Cons

- 进程是资源分配的最小单位，线程是程序执行的最小单位。
- 进程有自己的独立地址空间，每启动一个进程，系统就会为它分配地址空间，建立数据表来维护代码段、堆栈段和数据段，这种操作非常昂贵。而线程是共享进程中的数据的，使用相同的地址空间，因此CPU切换一个线程的花费远比进程要小很多，同时创建一个线程的开销也比进程要小很多。
- 线程之间的通信更方便，同一进程下的线程共享全局变量、静态变量等数据，而进程之间的通信需要以通信的方式（IPC)进行。不过如何处理好同步与互斥是编写多线程程序的难点。
- 但是多进程程序更健壮，多线程程序只要有一个线程死掉，整个进程也死掉了，而一个进程死掉并不会对另外一个进程造成影响，因为进程有自己独立的地址空间。

- *fork is expensive. Memory is copied from the parent to the child, all
  descriptors are duplicated in the child, and so on. Current
  implementations use a technique called copy-on-write, which avoids a
  copy of the parent's data space to the child until the child needs its
  own copy. But, regardless of this optimization, fork is expensive.*

- *IPC is required to pass information between the parent and child
  after the fork. Passing information from the parent to the child before
  the fork is easy, since the child starts with a copy of the parent's
  data space and with a copy of all the parent's descriptors. But,
  returning information from the child to the parent takes more work.*

- *Threads help with both problems. Threads are sometimes called
  lightweight processes since a thread is "lighter weight" than a process.
  That is, thread creation can be 10–100 times faster than process
  creation.*

- *All threads within a process share the same global memory. This makes
  the sharing of information easy between the threads, but along with
  this simplicity comes the problem*

## 协程和线程区别

协程避免了无意义的调度，由此可以提高性能，但程序员必须自己承担调度的责任。同时，协程也失去了标准线程使用多CPU的能力。

###线程

相对独立

有自己的上下文（Context）

切换受系统控制；

###协程

相对独立

有自己的上下文（Context）

切换由自己控制，由当前协程切换到其他协程由当前协程来控制。



##何时使用多进程，何时使用多线程?

对资源的管理和保护要求高，不限制开销和效率时，使用多进程。

要求效率高，频繁切换时，资源的保护管理要求不是很高时，使用多线程。

###为什么会有线程?

每个进程都有自己的地址空间，即进程空间，在网络或多用户环境下，一个服务器通常需要接收大量不确定数量用户的并发请求，为每一个请求都创建一个进程显然行不通（系统开销大响应用户请求效率低），因此操作系统中线程概念被引进。



##进程通信方式（选读）

###管道

速度慢，容量有限，只有父子进程能通讯

###FIFO

任何进程间都能通讯，但速度慢

###消息队列

容量受到系统限制，且要注意第一次读的时候，要考虑上一次没有读完数据的问题

###信号量

不能传递复杂消息，只能用来同步

###共享内存区

能够很容易控制容量，速度快，但要保持同步，比如一个进程在写的时候，另一个进程要注意读写的问题，相当于线程中的线程安全，当然，共享内存区同样可以用作线程间通讯，不过没这个必要，线程间本来就已经共享了同一进程内的一块内存



# 虚拟内存（Virtual Memory）

程序间的内存数据是安全的互不影响的。同时计算机程序直接运行在物理内存上也导致了内存使用率较低，程序运行内存地址不确定，不同的运行顺序甚至会出错。此时在程序的执行过程中，已经存在着大量在物理内存和硬盘之间的数据交换过程。

﻿基于以上问题，那我们可以是不是考虑在物理内存之上增加一个中间层，让程序通过虚拟地址去间接的访问物理内存呢。通过虚拟内存，每个进程好像都可以独占内存一样，每个进程看到的内存都是一致的，这称为虚拟地址空间。

（这种思想在现在也用的很广泛，例如很多优秀的中间层：Nginx、Redis等等）



### ###分段与分页

进程是操作系统资源分配的最小单元。操作系统分配给进程的内存空间中包含五种段：数据段、代码段、BSS、堆、栈。

- 数据段：存放程序中的静态变量和已初始化且不为零的全局变量。
- 代码段：存放可执行文件的操作指令，代码段是只读的，不可进行写操作。这部分的区域在运行前已知其大小。
- BSS段( Block Started By Symbol)：存放未初始化的全局变量，在变量使用前由运行时初始化为零。
- 堆：存放进程运行中被动态分配的内存，其大小不固定。
- 栈：存放程序中的临时的局部变量和函数的参数值