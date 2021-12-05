https://blog.csdn.net/niunai112/article/details/81071438

![a1d208e94dced5a3c52044dac34900ef.png](https://img-blog.csdnimg.cn/img_convert/a1d208e94dced5a3c52044dac34900ef.png)

##堆内存分配区域
###1、新生代(Young Generation)

几乎所有新生成的对象首先都放在新生代的。新生代内存按照8:1:1的比例分为一个Eden区和两个Survivor(Survivor0，Survivor1)区。大部分对象在Eden区中生成。当新对象生成而Eden Space申请失败(因为空间不足等)时，则会发起一起GC(Scavenge GC)。回收时先将Eden区存活对象复制到Survivor0区，然后清空Eden区，当这个Survivor0区也存放满了时，则将Eden区和Survivor0区存活对象复制到Survivor1区中，然后清空Eden区和Survivor0区，此时Survivor0区是空的，然后将Survivor0区和Survivor1区交换，即保持Survivor1为空，如此往复。当Survivor1区不足以存放Eden和Survivor0的存活对象时，就将存活对象直接放到老年代。当对象在Survivor区躲过一次GC的话，其对象年龄便会加1，默认情况下，如果对象年龄达到15岁，就会移动到老年代中。若是老年代也满了就会触发一次Full GC，也就是新生代、老年代都进行回收。新生代大小可以由-Xmn来控制，也可以用-XX：SurvivorRatio来控制Eden和Survivor的比例。

###2、年老代(Old Generation)

在年轻代中经历了N次垃圾回收后仍存活的对象，会被放到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象。内存比新生代也大很多(大概比例是1:2)，当老年代内存满时触发Major GC即Full GC，Full GC发生频率比较低，老年代对象存活时间比较长，存活率标记高。一般来说，大对象会被直接分配到老年代。所谓的大对象是指需要大量连续存储空间的对象，最常见的一种大对象是大数组。比如：

byte[] data = new byte[4*1024*1024]
这种一般会直接在老年代分配存储空间。

当然分配的规则并不是百分百固定的，这要取决于当前使用的是哪种垃圾收集器组合和JVM的相关参数。

###3、持久代(Permanent Generation)

用于存放静态文件(class类，方法)和常量等。持久代对垃圾回收没有显著的影响，但是有些应用可能动态生成或者调用一些class，例如Hibernate等，在这种时候需要设置一个比较大的持久空间来存放这些运行过程中新增的类。对永久代的回收主要回收两部分内容：废弃常量和无用的类。

永久代空间在Java SE8特性中已经被废除。取而代之的是元空间(MetaSpace)。因此不会出现“java.lang.OutOfMemoryError:PermGen error”错误。



## 对垃圾回收机制说明以下三点
1、新生代GC(Minor GC/Scavenge GC)：发生在新生代的垃圾收集动作。因为Java对象大多具有朝生夕灭的特性，因此Minor GC非常频繁(不一定等Eden区满了才触发)，一般回收速度也比较快。在新生代中，每次垃圾收集时会发现有大量对象死去，只有少量存活，因此可选用复制算法来完成收集。

2、老年代GC(Major GC/Full GC)：发生在老年代的垃圾回收动作。Major GC经常会伴随至少一次Minor GC。由于老年代中的对象生命周期比较长，因此Major GC并不频繁，一般都是等待老年代满了后才Full GC，而且其速度一般会比Minor GC慢10倍以上。另外，如果分配了Direct Memory，在老年代中进行Full GC时，会顺便清理掉Direct Memory中的废弃对象。而老年代中因为存活率高、没有额外空间对它进行分配担保，就必须使用标记-清除算法或标记-整理算法来进行回收

3、新生代采用空闲指针的方式来控制GC触发，指针保持最后一个分配的对象在新生代区间的位置，当有新的对象要分配内存时，用于检查空间是否足够，不够就触发GC。当连续分配对象时，对象会逐渐从Eden到Survivor，最后到老年代。

用Java VisualVM来查看，能明显观察到新生代满了后，会把对象转移到旧生代，然后清空继续装载，当老年代也满了后，就会报OutOfMemory的异常，如下图所示：

![cc14e66592a60c92756f4e9a0240f366.png](https://img-blog.csdnimg.cn/img_convert/cc14e66592a60c92756f4e9a0240f366.png)

**Partial GC**：并不收集整个GC堆的模式

- Young GC：只收集young gen的GC
- Old GC：只收集old gen的GC。只有CMS的concurrent collection是这个模式
- Mixed GC：收集整个young gen以及部分old gen的GC。只有G1有这个模式

**Full GC**：收集整个堆，包括young gen、old gen、perm gen（如果存在的话）等所有部分的模式。

 Minor GC ，Full GC 触发条件

Minor GC触发条件：当Eden区满时，触发Minor GC。

##Full GC触发条件：

（1）调用System.gc时，系统建议执行Full GC，但是不必然执行

（2）老年代空间不足

（3）方法去空间不足

（4）通过Minor GC后进入老年代的平均大小大于老年代的可用内存

（5）由Eden区、From Space区向To Space区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小


## 1.如何判定是否需要回收

　JVM要实现自动回收垃圾，那么它就需要判断，哪些内存可以回收，哪些不行。下面两个算法是虚拟机用来判断的方法

### 引用计数算法

正如算法名，这个算法就是给对象增加一个引用计数，每当对象被别的对象引用时，就将该对象的引用计数加一。所以当一个对象的引用计数为0的话，那么就说明这个对象没有被任何对象使用，那么JVM就可以认为这个对象是可以回收的对象啦。
　这个算法的优点是它的效率非常高，能非常直观的判断一个对象是否能被回收。

![reference](https://img-blog.csdn.net/20180719102357401?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pdW5haTExMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

　这图上的E对象就要被回收了（没有被任何对象引用）
　缺点也很明显，1.无法区分循环引用的对象（A引用了B，B引用了A），这2个对象的引用计数永远不可能为0，这2个对象无法被JVM回收。2.需要维护对象引用计数的值

### 可达性算法

为了解决引用计数算法的缺点，所以就有了可达性算法，这个算法就是通过 GC Roots 的对象作为起始点，然后通过这个节点往下找他引用的对象，直到最外层的叶子节点。当一个对象无法被 GC Roots 找到时，那么它就是可回收对象。

![GC Root](https://img-blog.csdn.net/20180719102642207?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pdW5haTExMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

这图上的E对象就要被回收了（E对象不可达）

**GC Roots对象的包括如下几种：**
1).虚拟机栈中的本地变量表中的引用的对象
2).方法区中的类静态属性引用的对象
3).方法区中的常量引用的对象
4).本地方法栈中JNI(Native Interface)的引用的对象



## 2.内存回收算法
　当JVM已经找到要回收的对象的时候就要开始回收对象了，JVM回收对象有以下三大回收算法

### 标记清除算法

这是最基础的垃圾回收算法，之所以说它是最基础的是因为它最容易实现，思想也是最简单的。标记-清除算法分为两个阶段：标记阶段和清除阶段。标记阶段的任务是标记出所有需要被回收的对象，清除阶段就是回收被标记的对象所占用的空间。
![marksweep](https://img-blog.csdn.net/20180719100936880?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pdW5haTExMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

　该算法会产生大量的内存碎片，可能会导致当JVM要分配大对象内存时，不能找到可用的内存空间的问题。

### 复制

　将内存划分为等大小的两块。每次只使用其中一块，当这一块内存满后将尚存活的对象复制到另一块上去，然后把满的那块内存清掉。

![copying](https://img-blog.csdn.net/20180719100912743?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pdW5haTExMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

　不会产生内存碎片了，但是可利用的内存将变成原来内存的一半，而且需要付出复制内存对象带来的消耗。

### 标记压缩

　结合了以上两个算法，为了避免缺陷而提出。先找出存活对象，然后把存活的对象移向内存的一端。然后清除端边界外的对象。
　![markcompact](https://img-blog.csdn.net/20180719100956570?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pdW5haTExMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

###分代收集算法

　分代收集法是目前大部分JVM所采用的方法，其核心思想是根据对象存活的不同生命周期将内存划分为不同的域，一般情况下将GC堆划分为新生代，老生代，永久代（元数据区）。因为不同的区域，其存储对象的特点不同，因此可以根据不同区域选择不同的算法。新生代的特点是每次垃圾回收时都有大量垃圾需要被回收，回收频率很高，老生代的特点是每次垃圾回收时只有少量对象需要被回收，回收频率很低。

## 3.垃圾回收器

### 年轻代垃圾回收器

#### serial

　单线程回收垃圾，它在进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集完成。
Serial收集器依然是虚拟机运行在Client模式下默认新生代收集器，对于运行在Client模式下的虚拟机来说是一个很好的选择。

![serial](https://img-blog.csdn.net/20180718155748337?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pdW5haTExMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

###parallel new
　ParNew收集器其实就是Serial收集器的多线程版本，除了使用多线程进行垃圾收集之外，其余行为包括Serial收集器可用的所有控制参数、收集算法、Stop The Worl、对象分配规则、回收策略等都与Serial 收集器完全一样。
　![img](https://img-blog.csdn.net/20180718155835156?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pdW5haTExMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

　ParNew收集器是许多运行在Server模式下的虚拟机中首选新生代收集器，其中有一个与性能无关但很重要的原因是，除Serial收集器之外，目前只有ParNew它能与CMS收集器配合工作

### parallel scavenge

　Parallel Scavenge收集器是一个新生代收集器，它也是使用复制算法的收集器，是并行的多线程收集器

　该收集器的目标是达到一个可控制的吞吐量（Throughput）。所谓吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值，即 吞吐量=运行用户代码时间/（运行用户代码时间+垃圾收集时间）

　停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验，而高吞吐量则可用高效率地利用CPU时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。

　Parallel Scavenge收集器提供两个参数用于精确控制吞吐量，分别是控制最大垃圾收起停顿时间的

　-XX:MaxGCPauseMillis参数以及直接设置吞吐量大小的-XX:GCTimeRatio参数

　Parallel Scavenge收集器还有一个参数：-XX:+UseAdaptiveSizePolicy。这是一个开关参数，当这个参数打开后，就不需要手工指定新生代的大小（-Xmn）、Eden与Survivor区的比例（-XX:SurvivorRatio）、晋升老年代对象年龄（-XX:PretenureSizeThreshold）等细节参数，只需要把基本的内存数据设置好（如-Xmx设置最大堆），然后使用MaxGCPauseMillis参数或GCTimeRation参数给虚拟机设立一个优化目标。

　自适应调节策略也是Parallel Scavenge收集器与ParNew收集器的一个重要区别

### 老年代垃圾回收器

#### serial old

老年代的垃圾回收器，它是一个单线程收集器，使用标记整理算法。

![serial old](https://img-blog.csdn.net/20180718155748337?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pdW5haTExMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


主要两大用途：

（1）在JDK1.5以及之前的版本中与Parallel Scavenge收集器搭配使用

（2）作为CMS收集器的后备预案，在并发收集发生Concurrent Mode Failure时使用

Serial Old收集器的工作工程

### parallel old

Parallel Old 是Parallel Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法。
![parallel old](https://img-blog.csdn.net/20180718155859166?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pdW5haTExMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### concurrent mark sweep

CMS(Concurrent Mark Sweep)收集器是一种以获取最短回收停顿时间为目标的收集器。目前很大一部分的Java应用集中在互联网站或者B/S系统的服务端上，这类应用尤其重视服务器的响应速度，希望系统停顿时间最短，以给用户带来较好的体验。CMS收集器就非常符合这类应用的需求

CMS从名字上就能看出，其是一个基于“标记-清除”算法实现的回收器，它的运作过程可以分为四个步骤： 1. 初始标记 2. 并发标记 3. 重新标记 4. 并发清除

CMS执行过程中对Java应用程序的影响如图所示：



![img](https://pic4.zhimg.com/80/v2-61dede309c89e5779cc9240daa927df3_1440w.jpg)



从图中我们可以看出，初始标记以及重新标记两个步骤仍然需要“Stop The World”。初始标记仅仅只是标记一下GC Roots能直接关联到的对象，速度很快，并发标记就是进行GC Roots Tracing的过程，而重新标记阶段是为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这一阶段的停顿时间一般会比初始标记阶段时间稍长一些，但远比并发标记的时间短。

由于整个过程中耗时最长的并发标记和并发清除过程回收器线程都可以与用户线程一起工作，因此，从总体上来说，CMS回收器的内存回收过程是与用户线程一起并发执行的。

CMS收集器主要优点：并发收集，低停顿。

CMS的缺点：

（1）CMS回收器采用的基础算法是Mark-Sweep。所有CMS不会整理、压缩堆空间。这样就会有一个问题：经过CMS收集的堆会产生空间碎片。 CMS不对堆空间整理压缩节约了垃圾回收的停顿时间，但也带来的堆空间的浪费。为了解决堆空间浪费问题，CMS回收器不再采用简单的指针指向一块可用堆空 间来为下次对象分配使用。而是把一些未分配的空间汇总成一个列表，当JVM分配对象空间的时候，会搜索这个列表找到足够大的空间来存下这个对象。

（2）CMS的另一个缺点是它需要更大的堆空间。因为CMS标记阶段应用程序的线程还是在执行的，那么就会有堆空间继续分配的情况，为了保证在CMS回 收完堆之前还有空间分配给正在运行的应用程序，必须预留一部分空间。也就是说，CMS不会在老年代满的时候才开始收集。相反，它会尝试更早的开始收集，已 避免上面提到的情况：在回收完成之前，堆没有足够空间分配！在JDK1.6中，CMS收集器的启动阀值已经提升至92%。CMS就开始行动了。 – XX:CMSInitiatingOccupancyFraction =n 来设置这个阀值。

（3）需要更多的CPU资源。从上面的图可以看到，为了让应用程序不停顿，CMS线程和应用程序线程并发执行，这样就需要有更多的CPU，单纯靠线程切 换是不靠谱的。并且，重新标记阶段，为空保证STW快速完成，也要用到更多的甚至所有的CPU资源。

(4) Floating garage, 在并行的时候因为多线程程会有新的垃圾产生

### G1 [JDK 1.9中，G1也成为了Server模式的默认回收器](https://link.zhihu.com/?target=http%3A//openjdk.java.net/jeps/248)。

JAVA8之后广泛使用，G1 将整个对区域划分为若干个Region，每个Region的大小是2的倍数（1M,2M,4M,8M,16M,32M，通过设置堆的大小和Region数量计算得出。
Region区域划分与其他收集类似，不同的是单独将大对象分配到了单独的region中，会分配一组连续的Region区域（Humongous start 和 humonous Contoinue 组成），所以一共有四类Region（Eden，Survior，Humongous和Old）， 

G1除了追求低停顿外，还有一个很典型的特征就是它能建立可预测的停顿时间模型。也就是说它能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾回收的时间不得超过N毫秒，该特性已经是实时回收器的特征了。 G1会尽最大努力来达成目标，并不能完全保证，一般称G1为*soft real-time collector*。

G1 作用于整个堆内存区域，设计的目的就是减少Full GC的产生。

### G1 内存结构

正如刚刚所说，G1不需要将堆分为连续的新生代和老年代，而是将其分成若干个（通常为2048个）较小的堆区域。每个区域可以是Eden区域，Survivor区域或者Old区域。所有Eden区域和Survivor区域是逻辑上的新生代，Old区域就是逻辑上的老年代，新生代和老年代在物理上不一定连续。G1的堆内存结构如下图所示：



![img](https://pic1.zhimg.com/80/v2-3a45644bdcc6309fac2ff951904d4cb0_1440w.jpg)

### Evacuation Pause: Fully Young

在应用程序刚刚执行时，由于没有任何回收经验信息，因此它最初以full-young的模式运行。当新生代填满时，应用程序线程停止，新生代内的数据会被复制到Survivor区域，此时其他目前空余的Region会成为新的Survivor。

这个复制的过程被称为“Evacuation”（疏散），它的工作方式与之前介绍的其他新生代回收方式（Eden Survivor 标记复制算法）几乎相同。

### Concurrent Marking

Full-Young模式之后，当堆的占用率超过45%（默认值，可以通过initiatingheapoccountypercent JVM参数进行修改）时，会开始进行表发标记过程。

不考虑Remembered Set的维护过程，G1中内存回收过程和CMS类似，前三个步骤和CMS回收几乎完全一样

需要注意的是：在筛选回收阶段，G1首先会对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间来制定回收计划。该阶段可以做到与用户程序一起并发执行

### 优势与不足

G1的特点有以下几点：

1. 并行与并发

能充分利用多CPU多核硬件优势，使用对个CPU（或CPU核心）来缩短STW的停顿时间。部分其他回收器原本需要停顿用户Java线程的时候，G1可以通过并发的方式让用户程序得以继续运行。

2. 对整个堆进行收集（仍有分代概念）

3. 空间整合（对比CMS）

对于两个Region之间，采用标记-复制算法， 不会产生空间碎片。有利于程序长时间运行、分配大对象时不会提前触发下一次FGC。

\4. 可预测的停顿（对比CMS）

可以说这是G1相比CMS的另一大优势，它能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾回收的时间不得超过N毫秒，该特性已经是实时回收器的特征了。

但是G1也有其不足，首先是设计更复杂，带来的就是更多的后台存活进程，对于吞吐量的开销就更大。因此，如果应用程序是吞吐量受限的，或者正在消耗100%的CPU，并且不太关心单个暂停的持续时间，那么CMS甚至Parallel可能是更好的选择。

### 各收集器间的搭配关系

![workinwith](https://img-blog.csdn.net/20180719114127980?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pdW5haTExMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)





![img](https://upload-images.jianshu.io/upload_images/3789193-1be27c1633ad2eeb.png?imageMogr2/auto-orient/strip|imageView2/2/w/612/format/webp)

##5.finalized
　finalize()的功能 : 一旦垃圾回收器准备释放对象所占的内存空间, 如果对象覆盖了finalize()并且函数体内不能是空的, 就会首先调用对象的finalize(), 然后在下一次垃圾回收动作发生的时候真正收回对象所占的空间.

　finalize()有一个特点就是: JVM始终只调用一次. 无论这个对象被垃圾回收器标记为什么状态, finalize()始终只调用一次. 但是程序员在代码中主动调用的不记录在这之内.

###finalize()主要使用的方面:

　根据垃圾回收器的第2点可知, java垃圾回收器只能回收创建在堆中的java对象, 而对于不是这种方式创建的对象则没有方法处理, 这就需要使用finalize()对这部分对象所占的资源进行释放. 使用到这一点的就是JNI本地对象, 通过JNI来调用本地方法创建的对象只能通过finalize()保证使用之后进行销毁,释放内存
充当保证使用之后释放资源的最后一道屏障, 比如使用数据库连接之后未断开,并且由于程序员的个人原因忘记了释放连接, 这时就只能依靠finalize()函数来释放资源.

###尽量避免使用finalize():
　finalize()不一定会被调用, 因为java的垃圾回收器的特性就决定了它不一定会被调用
就算finalize()函数被调用, 它被调用的时间充满了不确定性, 因为程序中其他线程的优先级远远高于执行finalize（）函数线程的优先级。也许等到finalize()被调用, 数据库的连接池或者文件句柄早就耗尽了.
如果一种未被捕获的异常在使用finalize方法时被抛出，这个异常不会被捕获，finalize方法的终结过程也会终止，造成对象出于破坏的状态。被破坏的对象又很可能导致部分资源无法被回收, 造成浪费.
finalize()和垃圾回收器的运行本身就要耗费资源, 也许会导致程序的暂时停止.





## CPU 排查

* `top` 查找cpu使用率高的进程
*  `top -H -p`  查找该进程内的线程

一般超过80%就是比较高的，80%左右是合理情况。这样我们就能得到CPU消耗比较高的线程id。接着通过该线程id的十六进制表示在jstack日志中查看当前线程具体的堆栈信息。

在这里我们就可以区分导致CPU过高的原因具体是Full GC次数过多还是代码中有比较耗时的计算了。如果是Full GC次数过多，那么通过jstack得到的线程信息会是类似于VM Thread之类的线程，而如果是代码中有比较耗时的计算，那么我们得到的就是一个线程的具体堆栈信息。如下是一个代码中有比较耗时的计算，导致CPU过高的线程信息：

![img](https://img2020.cnblogs.com/other/1218593/202005/1218593-20200521083354772-1575840876.jpg)

这里可以看到，在请求UserController的时候，由于该Controller进行了一个比较耗时的调用，导致该线程的CPU一直处于100%。我们可以根据堆栈信息，直接定位到UserController的34行，查看代码中具体是什么原因导致计算量如此之高。

##JVM/GC排查 

* https://www.jianshu.com/p/51052aaac3be

* 自身线程占用率过高

  * jstack {线程id(16进制)}

  ```
  SYNOPSIS
  jstack [ option ] pid
  jstack [ option ] executable core
  jstack [ option ] [server-id@]remote-hostname-or-IP
  PARAMETERS
  Options are mutually exclusive. Option, if used, should follow immediately after the command name. See OPTIONS.
  
  pid
  process id for which the stack trace is to be printed. The process must be a Java process. To get a list of Java processes running on a machine, jps may be used.
  executable
  Java executable from which the core dump was produced.
  
  core
  core file for which the stack trace is to be printed.
  
  remote-hostname-or-IP
  remote debug server's (see jsadebugd) hostname or IP address.
  
  server-id
  optional unique id, if multiple debug servers are running on the same remote host.
  ```

  * 第三方System.gc();

  