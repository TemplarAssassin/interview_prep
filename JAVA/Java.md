# JAVA 



### java 有哪些新建对象的方法

1. a = new A()

2. reflection 运用反射手段，调用Java.lang.Class或者java.lang.reflect.Constructor类的newInstance()实例方法。

   * **使用Class类的newInstance方法**

     可以使用Class类的newInstance方法创建对象。这个newInstance方法调用无参的构造函数创建对象。

     ```java
     //创建方法1
     User user = (User)Class.forName("base.package.User").newInstance();　
     //创建方法2（用这个最好）
     User user = User.class.newInstance();
     ```

   * **使用Constructor类的newInstance方法**

     和Class类的newInstance方法很像， java.lang.reflect.Constructor类里也有一个newInstance方法可以创建对象。我们可以通过这个newInstance方法调用有参数的和私有的构造函数。	

     ```java
     Constructor<User> constructor = User.class.getConstructor();
     User user = constructor.newInstance();
     ```

   ​	这两种newInstance方法就是大家所说的反射。事实上Class的newInstance方法内部调用Constructor的newInstance方法.

3. 使用Clone方法

   无论何时我们调用一个对象的clone方法，jvm就会创建一个新的对象，将前面对象的内容全部拷贝进去。用clone方法创建对象并不会调用任何构造函数。
   要使用clone方法，我们需要先实现[Cloneable](https://docs.oracle.com/javase/7/docs/api/java/lang/Cloneable.html)接口并实现其定义的clone方法。

   ```java
   public class CloneTest implements Cloneable{
       private String name;  
       private int age; 
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       public int getAge() {
           return age;
       }
   
       public void setAge(int age) {
           this.age = age;
       }
   
       public CloneTest(String name, int age) {
           super();
           this.name = name;
           this.age = age;
       }
   
       public static void main(String[] args) {
           try {
               CloneTest cloneTest = new CloneTest("wangql",18);
               CloneTest copyClone = (CloneTest) cloneTest.clone();
               System.out.println("newclone:"+cloneTest.getName());
               System.out.println("copyClone:"+copyClone.getName());
           } catch (CloneNotSupportedException e) {
               e.printStackTrace();
           }
       }
   }
   ```

4. [Deserializable](https://docs.oracle.com/javase/7/docs/api/java/io/Serializable.html)

### clone 是浅拷贝还是深拷贝?两种拷贝有什么区别?

###  equals 重写为什么需要重写 hashcode 函数?如果不改写会怎么样?

###数组(ArrayList)和链表(LinkedList)

