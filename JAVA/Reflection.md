## JAVA反射机制

JAVA反射机制是在**运行状态中**，

对于任意一个类，都能够知道这个类的所有属性和方法；

对于任意一个对象，都能够调用它的任意一个属性和方法；

这种动态获取的信息以及动态调用对象的方法的功能称为 **java语言的反射机制。**



其次就是反射相关的API，只讲一些常用的，比如获取一个Class对象。Class.forName(完整类名)。通过Class对象获取类的构造方法，class.getConstructor。根据class对象获取类的方法，getMethod和getMethods。使用class对象创建一个对象，class.newInstance等。

最后可以说一下反射的优点和缺点，优点就是增加灵活性，可以在运行时动态获取对象实例。缺点是反射的效率很低，而且会破坏封装，通过反射可以访问类的私有方法，不安全。



![img](https://pic3.zhimg.com/80/v2-b70d144c97ee5236cffb399dbf415226_1440w.jpg)

## 使用场景

**IDE自动提示功能**，对象（提示：属性、方法）

不知道类或者对象的具体信息，应该使用反射来实现。

> 比如类的名称放在XML文件中，属性和属性值放在XML文件中，需要在运行时读取XML文件，动态获取类的信息。

## 原理

Java在编译之后会生成一个**class文件**，反射通过**字节码文件**找到其类中的方法和属性等

## 功能

![img](https://pic4.zhimg.com/80/v2-b118132781b6b6fb618d789852a2b96b_1440w.jpg)

## 关键类

![img](https://pic4.zhimg.com/80/v2-79d585d1a955d5ee200cba45da6c50ef_1440w.jpg)

## Class对象

**类型标识，**JVM中为每个对象都保留其类型标识信息。

包含类的所有信息

可以通过该对象获取到**构造方法，成员变量，成员方法和接口**等信息

获取方法：

- 通过字面量直接获取，例如XXX.class，不会触发类的初始化但XXX类已经被加载到方法区。
- 通过Object类的getClass方法，例如Object.getClass()。触发类的初始化
- 通过Class的静态方法，例如Class.forName()。触发类的初始化

![img](https://pic1.zhimg.com/80/v2-51e0e944cb11c97a6b900e23a6208e30_1440w.jpg)

## Field

**成员变量，**类中的属性对象。

通过Class类的getDeclaredField()或getDeclaredFields()方法获取

![img](https://pic4.zhimg.com/80/v2-ada93c81804da33a2fae7c845aaf213b_1440w.jpg)

> Field的方法主要分为两大类，即getXXX和setXXX

## Method

类中的方法对象。包括了静态方法和成员方法（包括抽象方法在内）。

通过invoke()来完成方法被动态调用的目的。

> 非静态变量，需要添加对象参数

![img](https://pic3.zhimg.com/80/v2-49eea919c24b051045973dd529d342c6_1440w.jpg)



![img](https://pic1.zhimg.com/80/v2-b6c7632cad6f19b06248181f9944ad90_1440w.jpg)

> setAccessible()方法不影响其他对象和原方法

## getDeclaredMethod

可以获取**指定方法名和参数**的方法对象 **Method**。

```text
@CallerSensitive
public Method getDeclaredMethod(String name, Class<?>... parameterTypes)
    throws NoSuchMethodException, SecurityException {
    checkMemberAccess(Member.DECLARED, Reflection.getCallerClass(), true);
    //从返回的方法列表里找到一个匹配名称和参数的方法对象。
    Method method = searchMethods(privateGetDeclaredMethods(false), name, parameterTypes);
    if (method == null) {
        throw new NoSuchMethodException(getName() + "." + name + argumentTypesToString(parameterTypes));
    }
    return method;
}
```

## privateGetDeclaredMethods

从缓存或JVM中获取该Class中申明的方法列表。

## searchMethods

从返回的方法列表里找到一个**匹配名称和参数**的方法对象。

如果找到一个匹配的**Method**，则重新copy一份返回，即**Method.copy()**方法。

## ReflectionData

用来缓存从JVM中读取类的如下属性数据。

## Constructor

构造函数。类的构造方法

getConstructor() ：获取匹配的构造方法

![img](https://pic3.zhimg.com/80/v2-54515df3e26dd9fa65768487021557c6_1440w.jpg)

步骤：

1. 先获取所有的constructors, 然后通过进行参数类型比较；
2. 找到匹配后，通过 ReflectionFactory，copy一份constructor返回；
3. 否则抛出 NoSuchMethodException;

## 父类/父接口

![img](https://pic3.zhimg.com/80/v2-9c919d5e400b93b8556e8f712beeaf32_1440w.jpg)

**优点**

通过反射，java可以动态的加载未知的外部配置对象，临时生成字节码进行加载使用，使代码更灵活，极大地提高应用的扩展性。



### 三、Java 反射效率低的原因

了解了反射的原理以后，我们来分析一下反射效率低的原因。

#### 1. Method#invoke 方法会对参数做封装和解封操作

我们可以看到，invoke 方法的参数是 Object[] 类型，也就是说，如果方法参数是简单类型的话，需要在此转化成 Object 类型，例如 long ,在 javac compile 的时候 用了Long.valueOf() 转型，也就大量了生成了Long 的 Object, 同时 传入的参数是Object[]数值,那还需要额外封装object数组。
 而在上面 MethodAccessorGenerator#emitInvoke 方法里我们看到，生成的字节码时，会把参数数组拆解开来，把参数恢复到没有被 Object[] 包装前的样子，同时还要对参数做校验，这里就涉及到了解封操作。
 因此，在反射调用的时候，因为封装和解封，产生了额外的不必要的内存浪费，当调用次数达到一定量的时候，还会导致 GC。

#### 2. 需要检查方法可见性

通过上面的源码分析，我们会发现，反射时每次调用都必须检查方法的可见性（在 Method.invoke 里）

#### 3. 需要校验参数

反射时也必须检查每个实际参数与形式参数的类型匹配性（在NativeMethodAccessorImpl.invoke0 里或者生成的 Java 版 MethodAccessor.invoke 里）；

#### 4. 反射方法难以内联

Method#invoke 就像是个独木桥一样，各处的反射调用都要挤过去，在调用点上收集到的类型信息就会很乱，影响内联程序的判断，使得 Method.invoke() 自身难以被内联到调用方。参见 [www.iteye.com/blog/rednax…](https://www.iteye.com/blog/rednaxelafx-548536)

#### 5. JIT 无法优化

在 [JavaDoc](https://docs.oracle.com/javase/tutorial/reflect/index.html) 中提到：

> Because reflection involves types that are dynamically resolved, certain Java virtual machine optimizations can not be performed. Consequently, reflective operations have slower performance than their non-reflective counterparts, and should be avoided in sections of code which are called frequently in performance-sensitive applications.

因为反射涉及到动态加载的类型，所以无法进行优化。

### 总结

上面就是对反射原理和反射效率低的一些分析。


![img](https://i2.kknews.cc/SIG=f5n0dk/ctp-vzntr/5r42p02n92r04q6pn500np4sssq4s4q7.jpg)