# d

## 准备：

> 注意：在实际场景中，一般都是部署在Linux服务器中的项目报出内存溢出的问题；为了尽可能还原出实际场景，本文也是将提前编写好的可以触发内存溢出的代码并打包成可运行的Jar包，然后放到服务器中执行的。

#### 1、准备可导致内存溢出的代码：

```
// 创建一个Java类
public class OutOfMemory {
    
    private String test;
    
    public OutOfMemory(String test){
        this.test = test;
    }
    
}

// 模拟内存溢出的发生
public class TestOOM {
    
    public static void main(String[] args) {
        List<OutOfMemory> list = new ArrayList<OutOfMemory>();
        
        while(true){
            /**
             * 无限创建OutOfMemory对象，直至将堆空间占满，并且创建的OutOfMemory对象一直被list集合对象引用着，
             * 导致GC也无法回收，最终出现堆内存溢出问题
             */
            list.add(new OutOfMemory("5656"));
            System.out.println("5656");
        }
    }
}
```

> 代码编写完成后，使用开发工具导出**可运行的Jar包**（TestOOM.jar）

#### 2、准备Linux服务器

> 可以直接使用centos或者Red Hat等都可以；

## 实战：

#### 1、将可运行的Jar包放到服务器中执行：

①、可使用xshell、xftp工具将可运行的Jar包（Jar包叫：**TestOOM.jar**）放入到服务器中；
②、使用命令执行Jar包；命令：
   java **-Xms40m -Xmx70m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/usr/tmp**  -jar TestOOM.jar

注意：为了尽快模拟发生堆内存溢出，所以在启动Jar包时，设置了一些参数；参数解析：
   1）、 -Xms40m 初始堆大小设置为40m
   2）、 -Xmx70m 最大堆大小设置为70m
   3）、 -XX:+HeapDumpOnOutOfMemoryError 出现堆内存溢出时，自动导出堆内存 dump 快照
   4）、 -XX:HeapDumpPath=/usr/tmp 设置导出的堆内存快照的存放地址为 /usr/tmp

#### 2、执行成功后，使用JVM监控命令监控JVM的信息：

①、jps命令：此命令是用来查询与Java相关的进程的，并输出进程号；下图就是展示上面运行的Jar包的进程号：
![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL2xlaXNoZW42L0ltZ0hvc3RpbmcvTXVaaUxlaV9ibG9nX2ltZy8yMDIwMDUxNTExMTMxMC5wbmc?x-oss-process=image/format,png)

②、jmap命令： **jmap -heap 3324** 此命令是查询出进程号为 3324 的JVM中堆内存信息；如下图：
![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL2xlaXNoZW42L0ltZ0hvc3RpbmcvTXVaaUxlaV9ibG9nX2ltZy8yMDIwMDUxNTExMTMyMy5wbmc?x-oss-process=image/format,png)

> 在图中可以发现堆内存中新生代、年老代中 free 可用空间越来越小，这预示着即将会发生GC垃圾回收，从而使堆腾出更多的空间存放新创建的对象。

③、jstat命令：使用其监控JVM的性能信息；例如：在本次排查内存溢出的问题中，会使用 jstat 命令监控 JVM的 GC垃圾回收的情况；
 命令： **jstat -gcutil 3324 1000**  意思是每1000毫秒查询一次进程号为3324 的JVM的GC垃圾回收的情况；如下图：
![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL2xlaXNoZW42L0ltZ0hvc3RpbmcvTXVaaUxlaV9ibG9nX2ltZy8yMDIwMDUxNTExMTMxOC5wbmc?x-oss-process=image/format,png)

> （1）、 YGC(堆中新生代GC)、FGC( FULL GC)为什么触发频率这么快呢？
> 答：由于堆内存空间不够用了，需要通过GC垃圾回收将一些空间进行回收，用于存放新创建的对象。

> （ 2）、当堆内存空间不够用时，GC具体会发生什么呢？
>
> 答：
>
> 1）、当堆中的新生代空间不够用时，会触发YGC(又称之为 Minor GC)，对堆中新生代空间进行垃圾回收，在垃圾回收后剩余存活的对象会中，如果有对象的
>
>  
>
> “晋升年龄达“ 到 “晋升年龄阈值” 后
>
>  
>
> ，就会将其移动到堆中
>
> 老年代存储，所以每次YGC后，堆中年老代中存储的对象数量可能会增大；
>
> 注意：晋升年龄：指新生代中的对象每经历一次 YGC，晋升年龄加1；
>
> 注意：晋升年龄阈值：在Serial和ParNew GC两种回收器中，“晋升年龄阈值” 可通过参数MaxTenuringThreshold设定，默认值为15。
>
> 上面的 “晋升年龄” 来自：
>
>  
>
> 从实际案例聊聊Java应用的GC优化
>
> 2）、当堆的新生代即将发生YGC时，如果发现新生代中存活下来的对象中达到“晋升年龄阈值”的对象所占用的空间比堆中年老代中剩余的可用空间大的话，就会直接不进行YGC，而会直接触发FGC，FULL GC会对整个堆空间（新生代、老年代）以及方法区/永久代进行垃圾回收；
> **注意：FULL GC 主要可以分为两步，先是对 堆中老年代进行垃圾回收（又称之为Major GC），然后再对 堆中新生代进行垃圾回收（YGC）。**

**扩展：堆的结构图：**

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL2xlaXNoZW42L0ltZ0hvc3RpbmcvTXVaaUxlaV9ibG9nX2ltZy8yMDIwMDUxNTExMTQwNS5wbmc?x-oss-process=image/format,png)

#### **3、出现内存溢出后，会自动生成快照，然后分析堆内存快照：**

①、使用XFTP等工具将服务器中的快照文件导出，本文堆内存快照文件是以 **hprof** 为后缀的文件；导出快照文件后，可以通过JDK自带的 **jvisualvm.exe** 分析工具打开进行分析。

> jvisualvm.exe 是在哪里呢？（以 windows 系统为例）
> 它是在 **JDK的安装目录中的bin目录下的**。

**如图：**
![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL2xlaXNoZW42L0ltZ0hvc3RpbmcvTXVaaUxlaV9ibG9nX2ltZy8yMDIwMDUxNTExMTM1NC5wbmc?x-oss-process=image/format,png)

②、使用 jvisualvm.exe 导入快照文件，如图：
（1）、
![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL2xlaXNoZW42L0ltZ0hvc3RpbmcvTXVaaUxlaV9ibG9nX2ltZy8yMDIwMDUxNTE0NDMwMi5wbmc?x-oss-process=image/format,png)
（2）、
![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL2xlaXNoZW42L0ltZ0hvc3RpbmcvTXVaaUxlaV9ibG9nX2ltZy8yMDIwMDUxNTE0NDMxNi5wbmc?x-oss-process=image/format,png)
（3）、
![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL2xlaXNoZW42L0ltZ0hvc3RpbmcvTXVaaUxlaV9ibG9nX2ltZy8yMDIwMDUxNTE0NDMzNS5wbmc?x-oss-process=image/format,png)

> 通过分析堆内存快照得到的结论：
> 通过第（3）张图，可以发现堆内存中有一个实例对象的占比为 99.9%，可以确定是由于这个实例对象大量创建导致堆内存的溢出；
> 说到这，可以回过头去看下我们自己编写的可以触发堆内存溢出的小程序，发现正是由于在 **while(true)** 死循环 中无线创建 OutOfMemory对象，导致堆内存空间被耗尽。