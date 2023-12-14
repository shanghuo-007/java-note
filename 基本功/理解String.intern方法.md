# 理解String.intern()

### String默认值

`new String()`和`new String(“”)`都是申明一个新的空字符串，是空串不是`null`；

```java
        String s1 = new String();
        String s2 = new String("");
        String s3 = null;
        System.out.println(s1 == null);
        System.out.println(s2 == null);
        System.out.println(s3 == null);
```

输出：

```cmd
false
false
true
```

### 引入概念

在这里，我们不谈堆，也不谈栈，只先简单引入常量池这个简单的概念。 
**常量池(constant pool)**指的是在编译期被确定，并被保存在已编译的.class文件中的一些数据。它包括了关于类、方法、接口等中的常量，也包括字符串常量。 

### 直接声明字符串

看例1： 

```java
String s0=”kvill”; 
String s1=”kvill”; 
String s2=”kv” + “ill”; 
System.out.println( s0==s1 ); 
System.out.println( s0==s2 );
```

结果为： 

```cmd
true 
true
```

首先，我们要知道Java会确保一个字符串常量只有一个拷贝。 
因为例子中的s0和s1中的”kvill”都是字符串常量，它们在编译期就被确定了，所以`s0 == s1`为true；而”kv”和”ill”也都是字符串常量，当一个字符串由多个字符串常量连接而成时，它自己肯定也是字符串常量，所以s2也同样在编译期就被解析为一个字符串常量，所以s2也是常量池中”kvill”的一个引用。
所以我们得出`s0 == s1 == s2`; 

### 使用new String()
用new String() 创建的字符串不是常量，不能在编译期就确定，所以new String() 创建的字符串不放入常量池中，它们有自己的地址空间。

例2：

```java
1 String s0=”kvill”; 
2 String s1=new String(”kvill”); 
3 String s2=”kv” + new String(“ill”); 
4 System.out.println( s0==s1 ); 
5 System.out.println( s0==s2 ); 
6 System.out.println( s1==s2 );
```

结果为： 

```cmd
false 
false 
false
```

例2中s0还是常量池中”kvill”的应用，s1因为无法在编译期确定，所以是运行时创建的新对象”kvill”的引用，s2因为有后半部分new String(“ill”)所以也无法在编译期确定，所以也是一个新创建对象”kvill”的应用;明白了这些也就知道为何得出此结果了。 

### String.intern()

==存在于.class文件中的常量池，在运行期被JVM装载，并且可以扩充。String的intern()方法就是扩充常量池的一个方法；当一个String实例str调用intern()方法时，Java查找常量池中是否有相同Unicode的字符串常量，如果有，则返回其的引用，如果没有，则在常量池中增加一个Unicode等于str的字符串并返回它的引用==

```java
String s0= “kvill”; 
String s1=new String(”kvill”); 
String s2=new String(“kvill”); 
System.out.println( s0==s1 ); 
System.out.println( “**********” ); 
s1.intern(); 
s2=s2.intern(); //把常量池中“kvill”的引用赋给s2 
System.out.println( s0==s1); 
System.out.println( s0==s1.intern() ); 
System.out.println( s0==s2 );
```

结果为：

```cmd
false 
********** 
false //虽然执行了s1.intern(),但它的返回值没有赋给s1 
true //说明s1.intern()返回的是常量池中”kvill”的引用 
true
```







