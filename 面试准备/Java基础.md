# Java相关

## 基础

### 面相对象特性

继承、封装、多态

### a = a + b 与 a += b 的区别

1. 性能区别：a=a+b是加法运算 需要两次寻找地址而a+=b是增量运算有寄存器优先时 只有一次地址查找。
2. 数据类型方面：+= 隐式的将加操作的结果类型强制转换为持有结果的类型。如果两个整型相加，如 byte、short 或者 int，首先会将它们提升到 int 类型，然后在执行加法操作。

即前者会将a和b从short或byte提升到int类型，在赋值给a，此时会因为精度丢失导致报错。

### 3*0.1 == 0.3 将会返回什么? true 还是 false?

false，因为有些浮点数不能完全精确的表示出来。

### 能在 Switch 中使用 String 吗?

从 Java 7 开始，我们可以在 switch case 中使用字符串，但这仅仅是一个语法糖。**内部实现在 switch 中使用字符串的 hash code。**

### ==和equals

-  ==

   1. 如果是基本数据类型的比较，则比较的是值。

   2. 如果是包装类或者引用类的比较，则比较的是对象地址。

- equals

  1. 没有重写equals方法，那么比较的就是两个对象的地址，就是使用==来比较的。

  2. 重写了equals方法后，还得看equals方法是如何写的。典型的String/Integer等这些都是重写了eqauls方法的。

  在Integer中比较的是对应数字的值，在String中是把字符串拆成char数组进行一个一个比较。

  另外，直接使用`Integer i=128;`这行代码的时候，i被自动装箱成`Integer`，会执行`valueOf`方法：

  ![image-20211013165855166](https://gitee.com/Shanghuo11/picture/raw/master/img/202312021124379.png)

这里`i`的值如果在`-128 - 127`之间，则直接从缓存中加载直接赋值，而不是`new`一个`Integer`对象。

因此下面这行代码会返回`false`：

```java
Integer i1 = 128;
Integer i2 = 128;
boolean b1 = i1 == i2;		//false
boolean b2 = i1.equals(i2); //true
```



### 对hashCode()的理解?

hashCode() 的作用是获取哈希码，这个哈希码的作用是确定该对象在哈希表中的索引位置，更好的支持hash表，比如String，Set，HashTable、HashMap等。

若一个类重写了`equals`方法，却没有重写`hashCode`，可能会出现`equals`返回true，而`hashCode`结果却不同，这样就导致HashMap这样的集合无法正常使用。

### jdk13、jdk17新特性？

jdk13:

1. 动态CDS归档
2. 重新实现遗留套接字api
3. Switch表达式
4. 文本块
5. ZGC改进

jdk17:

1. 上下文特定的反序列化过滤器
2. jdk内部模块的强封装
3. 密封JNI
4. 改进垃圾收集器

### java面向对象的设计原则是什么？

面向对象七大设计原则：

1. 单一职责原则

   核心：解耦和增强内聚性（高内聚，低耦合）

2. 里氏替换原则

   核心：在任何父类出现的地方都可以用他的子类来替代

3. 依赖注入原则

   核心：要依赖于抽象，不要依赖于具体的实现

4. 接口分离原则

   核心：一个接口不需要提供太多的行为，一个接口应该只提供一种对外的功能，不应该把所有的操作都封装到一个接口当中

5. 开闭原则

   核心：对扩展开放，对修改关闭。即在设计一个模块的时候，应当使这个模块可以在不被修改的前提下被扩展。

6. 迪米特法则

   核心：一个对象应当对其他对象有尽可能少的了解。在模块之间只通过接口来通信，而不理会模块的内部工作原理，可以使各个模块的耦合成都降到最低，促进软件的复用。

7. 合成聚合复用原则

   核心：尽量使用对象组合，而不是继承来达到复用的目的。



## 泛型

### 什么是泛型

在定义一个类、接口或者方法时可以指定类型参数。这个类型参数我们可以在使用类、接口或者方法时动态指定。

### 如何理解Java中的泛型是伪泛型？

泛型中类型擦除 Java泛型这个特性是从JDK 1.5才开始加入的，因此为了兼容之前的版本，Java泛型的实现采取了“伪泛型”的策略，即Java在语法上支持泛型，但是在编译阶段会进行所谓的“类型擦除”（Type Erasure），将所有的泛型表示（尖括号中的内容）都替换为具体的类型（其对应的原生态类型），就像完全没有泛型一样。

举例：方法的重载，``test(List<Integer>)`` 与 `test(List<String>)` 无法重载。

泛型擦除是指Java中的泛型只在编译期有效，在运行期间会被删除。也就是说所有泛型参数在编译后都会被清除掉。



## 集合

### arrayList是怎么实现动态数组的

之所以是动态数组，是因为在使用无参构造函数创建ArrayList之后，默认为空数组

```java
public ArrayList() {  
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;  
}
```

在你每次去add一个元素的时候，通过**ensureCapacityInternal**方法去检查添加后元素的个数是否会超出当前数组的长度，如果超出，数组将会进行扩容，以满足添加数据的需求。

若add时ArrayList为空，会进行赋值初始容量10，后续超过初始容量后进行扩容，扩容后大小为原来的1.5倍。

```java
public boolean add(E e) {
	// size为当前数据存放元素的数量
    ensureCapacityInternal(size + 1);
    // elementData为ArrayList内部存放数据的数组
    elementData[size++] = e;  
    return true;
}

private void ensureCapacityInternal(int minCapacity) { 
	// 判断数组是否为空，为空则将最小容量设置为默认值10
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {  
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);  
    }  
	// 判断当前容量是否满足最小容量
    ensureExplicitCapacity(minCapacity);  
}

private void ensureExplicitCapacity(int minCapacity) {
	// modCount用于记录修改次数，作用于fast-fail机制
	// 如果在迭代的时候发现modCount被修改了，则会抛出异常来避免可能会出现的不确定行为
    modCount++;  
  
    // 如果当前容量小于最小容量，则进行扩容
    if (minCapacity - elementData.length > 0)  
        grow(minCapacity);  
}

private void grow(int minCapacity) {  
    int oldCapacity = elementData.length;  
    // 计算新的容量，扩大为原容量的1.5倍, oldCapacity >> 1 运算表示右移一位，也就是相当于除以二并取整
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)  
        newCapacity = minCapacity; 
    // 如果新容量超过了数组的最大容量,调用hugeCapacity方法来获取一个更大的容量
    if (newCapacity - MAX_ARRAY_SIZE > 0)  
        newCapacity = hugeCapacity(minCapacity);  
    // 拷贝原数组，将原数组的元素复制到新数组中 
    elementData = Arrays.copyOf(elementData, newCapacity);  
}

private static int hugeCapacity(int minCapacity) {  
    if (minCapacity < 0) 
        throw new OutOfMemoryError();
	// 如果最小容量大于最大容量，则将容量设置为Integer.MAX_VALUE
    return (minCapacity > MAX_ARRAY_SIZE) ?  
        Integer.MAX_VALUE :  
        MAX_ARRAY_SIZE;  
}
```

### hashMap存取数据的时间复杂度是多少

在理想状态下，及未发生任何hash碰撞，数组中的每一个链表都只有一个节点，那么get方法可以通过hash直接定位到目标元素在数组中的位置，时间复杂度为O(1)。

若发生hash碰撞，则可能需要进行遍历寻找，n个元素的情况下，链表时间复杂度为O(n)、红黑树为O(logn)

初始化HashMap，设置默认负载因子

```java
public HashMap() {
      this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
  }
```

存储数据方法：

```java
public V put(K key, V value) {
	return putVal(hash(key), key, value, false, true);
}

static final int hash(Object key) {
    int h;
  	//如果为null，则hash结果返回0；否则返回key的hashCode与key的hashCode的高16位进行异或运算
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
  		//如果初始hashMap为空，进行初始化resize();初始容量为16，
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
      //如果桶为空，则直接新增一个节点
        tab[i] = newNode(hash, key, value, null);
    else {
      //发生哈希碰撞，桶中存在其他元素
        Node<K,V> e; K k;
      	//如果两个元素想同，直接替换
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
          	//如果当前节点是树，放到树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
          	//链表尾部添加元素
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                  	//超过8个节点转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
  	//若果超过最大容量*负载因子，进行扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}

final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 超过最大值就不再扩充了，就只好随你碰撞去吧
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        // 创建对象时初始化容量大小放在threshold中，此时只需要将其作为新的数组容量
        newCap = oldThr;
    else {
        // signifies using defaults 无参构造函数创建的对象在这里计算容量和阈值
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        // 创建时指定了初始化容量或者负载因子，在这里进行阈值初始化，
    	// 或者扩容前的旧容量小于16，在这里计算新的resize上限
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    // 只有一个节点，直接计算元素新的位置即可
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    // 将红黑树拆分成2棵子树，如果子树节点数小于等于 UNTREEIFY_THRESHOLD（默认为 6），则将子树转换为链表。
                    // 如果子树节点数大于 UNTREEIFY_THRESHOLD，则保持子树的树结构。
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else {
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```







## 多线程

### volitail关键字的作用，和锁比起来性能如何

作用：

1. 保证修饰的属性具有可见性
2. 可以禁止指令重排序，从而实现了有序性

volatile的同步机制确实优于锁，（synchronized关键字或是JUC包中的锁）但是由于虚拟机进行了许多优化升级，我们也并不能认为volatile比锁快多少。

首先，volatile修饰的变量进行读操作与普通变量几乎没什么差别，但是写操作相对慢一些，因为它需要在本地代码中插入很多内存屏障来保证指令不会发生乱序执行，但是开销总是比锁要小。



## JVM相关

### jdk1.8默认的垃圾收集器是什么？使用的什么算法？

JDK 8 默认使用的垃圾回收算法是**并行垃圾回收器（Parallel Garbage Collector）**。该垃圾回收器采用了多线程的方式进行垃圾回收，可以在多核处理器上充分利用多线程的优势，提高垃圾回收的效率。

并行垃圾回收器分为两个阶段：新生代和老年代。即使用==分代收集算法==：在新生代中，采用了**复制算法**进行垃圾回收，即将存活的对象复制到一个干净的区域，并将原来的区域全部清空。在老年代中，采用了**标记-清除（Mark and Sweep）算法**进行垃圾回收，即首先标记所有存活的对象，然后清除所有未被标记的对象。

### JVM分区有哪些？哪些区域会发生OOM？



### 哪几种类加载器

启动类加载器、扩展类加载器、系统类加载器

### 双亲委派知道吗，流程，为什么要双亲委派？





### 父类静态代码块、父类构造方法、子类静态代码块、子类构造方法的执行顺序

父类静态代码块、子类静态代码块、父类构造方法、子类构造方法





## Spring相关

### Spring,Spring MVC,Spring Boot 之间什么关系?

Spring 包含了多个功能模块，其中最重要的是 Spring-Core（主要提供 IoC 依赖注入功能的支持） 模块， Spring 中的其他模块（比如 Spring MVC）的功能实现基本都需要依赖于该模块。

Spring MVC 是 Spring 中的一个很重要的模块，主要赋予 Spring 快速构建 MVC 架构的 Web 程序的能力。MVC 是模型(Model)、视图(View)、控制器(Controller)的简写，其核心思想是通过将业务逻辑、数据、显示分离来组织代码。

使用 Spring 进行开发各种配置过于麻烦比如开启某些 Spring 特性时，需要用 XML 或 Java 进行显式配置。于是，Spring Boot 诞生了！

Spring 旨在简化 J2EE 企业应用程序开发。Spring Boot 旨在简化 Spring 开发（减少配置文件，开箱即用！）。

Spring Boot 只是简化了配置，如果你需要构建 MVC 架构的 Web 程序，你还是需要使用 Spring MVC 作为 MVC 框架，只是说 Spring Boot 帮你简化了 Spring MVC 的很多配置，真正做到开箱即用！



### 谈谈自己对于 Spring IoC 的了解

>  **IoC（Inversion of Control:控制反转）** 是一种设计思想，而不是一个具体的技术实现。IoC 的思想就是将原本在程序中手动创建对象的控制权，交由 Spring 框架来管理。不过， IoC 并非 Spring 特有，在其他语言中也有应用。

**为什么叫控制反转？**

- **控制**：指的是对象创建（实例化、管理）的权力
- **反转**：控制权交给外部环境（Spring 框架、IoC 容器）

![IoC 图解](https://gitee.com/Shanghuo11/picture/raw/master/img/202401111423646.png)

将对象之间的相互依赖关系交给 IoC 容器来管理，并由 IoC 容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。 IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。

在实际项目中一个 Service 类可能依赖了很多其他的类，假如我们需要实例化这个 Service，你可能要每次都要搞清这个 Service 所有底层类的构造函数，这可能会把人逼疯。如果利用 IoC 的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。

在 Spring 中， IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个 Map（key，value），Map 中存放的是各种对象。



### @Autowire、@Qualifier 和 @Resource 之间的关系

`Autowired` 属于 Spring 内置的注解，默认的注入方式为`byType`（根据类型进行匹配），也就是说会优先根据接口类型去匹配并注入 Bean （接口的实现类）。当一个接口存在多个实现类时，注入方式会变为 `byName`（根据名称进行匹配）。

例：`SmsService` 接口有两个实现类: `SmsServiceImpl1`和 `SmsServiceImpl2`，且它们都已经被 Spring 容器所管理。

```java
// 报错，byName 和 byType 都无法匹配到 bean
@Autowired
private SmsService smsService;
// 正确注入 SmsServiceImpl1 对象对应的 bean
@Autowired
private SmsService smsServiceImpl1;
// 正确注入  SmsServiceImpl1 对象对应的 bean
// smsServiceImpl1 就是我们上面所说的名称
@Autowired
@Qualifier(value = "smsServiceImpl1")
private SmsService smsService;
```

@Qualifier 与 @Autowire 搭配使用，来**显式指定名称**而不是依赖变量的名称。

`@Resource`属于 JDK 提供的注解，默认注入方式为 `byName`。如果无法通过名称匹配到对应的 Bean 的话，注入方式会变为`byType`

```java
// 报错，byName 和 byType 都无法匹配到 bean
@Resource
private SmsService smsService;
// 正确注入 SmsServiceImpl1 对象对应的 bean
@Resource
private SmsService smsServiceImpl1;
// 正确注入 SmsServiceImpl1 对象对应的 bean（比较推荐这种方式）
@Resource(name = "smsServiceImpl1")
private SmsService smsService;
```

总结：

- `@Autowired` 是 Spring 提供的注解，`@Resource` 是 JDK 提供的注解。
- `Autowired` 默认的注入方式为`byType`（根据类型进行匹配），`@Resource`默认注入方式为 `byName`（根据名称进行匹配）。
- 当一个接口存在多个实现类的情况下，`@Autowired` 和`@Resource`都需要通过名称才能正确匹配到对应的 Bean。`Autowired` 可以通过 `@Qualifier` 注解来显式指定名称，`@Resource`可以通过 `name` 属性来显式指定名称。
- `@Autowired` 支持在构造函数、方法、字段和参数上使用。`@Resource` 主要用于字段和方法上的注入，不支持在构造函数或参数上使用。



### Bean的生命周期

![Spring Bean 生命周期](https://gitee.com/Shanghuo11/picture/raw/master/img/202401111511379.jpg)

### spring bean的初始化和实例化的区别

- 实例化：是对象创建的过程。比如使用构造方法new对象，为对象在内存中分配空间。

- 初始化：是为对象中的属性赋值的过程。





## SpringBoot 相关

### 在springboot启动阶段或者初始阶段去获取外部的配置数据来启动项目，怎么去实现



### 你会怎么去开发一个starter，运用了springboot的什么机制



### 讲讲引入一个mysql的starter或者kafka的starter，他是怎么去加载的？启动原理是什么？









## SQL相关

### 数据库建表的三大范式

1. 确保每列保持原子性
2. 确保表中的每列都和主键相关
3. 确保每列都和主键列直接相关,而不是间接相关

### 一对多和多对多情况下表设计

**一对多**：一对多可以建两张表，将一这一方的主键作为多那一方的外键，例如一个学生表可以加一个字段指向班级

**多对多**：多对多可以多加一张中间表，将另外两个表的主键放到这个表中（如学生和课程就是多对多的关系）

多对多使用中间表符合数据库三大范式，另外中间表一般是作为关系表存在，如果改关系失效，仅删除关系，无需操作实体表数据。

### union 和 union  all的区别是什么

UNION和UNION ALL关键字都是将两个结果集合并为一个，但这两者从使用和效率上来说都有所不同。

1、对重复结果的处理：UNION在进行表链接后会筛选掉重复的记录，Union All不会去除重复记录。

2、对排序的处理：Union将会按照字段的顺序进行排序；UNION ALL只是简单的将两个结果合并后就返回。

从效率上说，UNION ALL 要比UNION快很多，所以，如果可以确认合并的两个结果集中不包含重复数据且不需要排序时的话，那么就使用UNION ALL。

- 注意：两个要联合的SQL语句**字段个数**必须一样，而且**字段类型**要“相容”（一致）；



### 左连接和右链接的区别

左连接是以左表为参照，显示所有数据；右链接，以右表为参照显示数据；



### 怎么优化一个慢SQL

![image-20240110161840124](https://gitee.com/Shanghuo11/picture/raw/master/img/202401101618295.png)

1. 找到慢SQL，默认情况下MySQL数据库不启动慢查询日志，需要手动将参数设置为：ON。

2. MySQL提供了一个explain命令, 它可以对`select`语句进行分析, 并输出`select`执行的详细信息。

   通过explain命令，我们能分析出一些慢SQL的常见原因：

   - 索引使用问题，通过 `possible_keys(能用到的索引)`和 `key(实际用到的索引)`两个字段查看：

     - 没有使用索引

     - 优化器选择了错误索引

     - 没有实现覆盖索引

   - I/O开销问题，通过``rows(执行当前查询要遍历的行数)``和``filtered(有效行数/扫描行数比值)`字段来查看：

     - 扫描的行数过多
     - 返回无用列且无用列有明显I/O性能开销(比如text、blob、json 等类型）

3. explain只能分析到SQL的预估执行计划，无法分析到SQL实际执行过程中的耗时，可以通过配置profiling参数来进行SQL执行分析。开启参数后，后续执行的SQL语句都会记录其资源开销，包括IO，上下文切换，CPU，Memory等等，我们可以根据这些开销进一步分析当前慢SQL的瓶颈再进一步进行优化。

4. profile只能查看到SQL的执行耗时，但是无法看到SQL真正执行的过程信息。Optimizer Trace是一个跟踪功能，它可以跟踪执行语句的解析优化执行的全过程，可以开启该功能进行执行语句的分析。 诚如上述所说，explain只能判断出SQL预估的执行计划，预估时根据成本模型进行成本计算进而比较出理论最优执行计划，但在实际过程中，预估不代表完全正确，因此我们需要通过追踪来看到它在执行过程中的每一环是否真正准确。

### 如何判断走没走索引

使用Explain

### Explain用法

在查询sql前面加一个explain

| 列名          | 含义                                                         |
| ------------- | ------------------------------------------------------------ |
| select_type   | select子句类型  simple primary subquery derived union union result |
| partitions    | 匹配的分区                                                   |
| type          | 访问类型，即怎么找数据行的方式(ALL, index, range, ref, eq_ref, const, system, NULL) |
| possible_keys | 此次查询中涉及字段上若存在索引，则会被列出来，表示可能会用到的索引，但并不是实际上一定会用到的索引 |
| key           | 此次查询中实际上用到的索引                                   |
| key_len       | 索引使用的字节数                                             |
| ref           | 连接匹配条件                                                 |
| rows          | 根据表信息统计以及索引的使用情况，大致估算说要找到所需记录需要读取的行数，rows 越小越好 |
| filtered      | 通过条件过滤出的行数所占百分比估计值，1~100，100表示没有做任何过滤 |
| Extra         | 该列包含MySQL解决查询的详细信息                              |

type 是代表 MySQL 使用了哪种索引类型，不同的索引类型的查询效率也是不一样的。

- system:表中只有一行记录，system 是 const 的特例，几乎不会出现这种情况，可以忽略不计
- const:将主键索引或者唯一索引放到 where 条件中查询，MySQL 可以将查询条件转变成一个常量，只匹配一行数据，索引一次就找到数据了
- eq_ref:在多表查询中，如 T1 和 T2，T1 中的一行记录，在 T2 中也只能找到唯一的一行，说白了就是 T1 和 T2 关联查询的条件都是主键索引或者唯一索引，这样才能保证 T1 每一行记录只对应 T2 的一行记录
- ref:不是主键索引，也不是唯一索引，就是普通的索引，可能会返回多个符合条件的行。
- range:体现在对某个索引进行区间范围检索，一般出现在 where 条件中的 between、and、<、>、in 等范围查找中
- index:将所有的索引树都遍历一遍，查找到符合条件的行。索引文件比数据文件还是要小很多，所以比不用索引全表扫描还是要快很多。
- all:没用到索引，单纯的将表数据全部都遍历一遍，查找到符合条件的数据



### 索引失效的场景

1. 联合索引不满足最左匹配原则

​	即对索引使用左或者左右模糊匹配，如`select * from t_user where name like '%林';`

​	**失效原因**：MySQL采用B+树，索引 B+ 树是按照「索引值」有序排列存储的，只能根据前缀进行比较。

​	**例外**：如果数据库表中的字段只有主键+二级索引，那么即使使用了左模糊匹配，也不会走全表扫描（type=all），而是走全扫描二级索引树(type=index)。

2. 使用函数导致索引失效

​	例：`select * from t_user where length(name)=6;`

​	因为索引保存的是索引字段的原始值，而不是经过函数计算后的值，自然就没办法走索引了。

不过，从 MySQL 8.0 开始，索引特性增加了函数索引，即可以针对函数计算后的值建立一个索引，也就是说该索引的值是函数计算后的值，所以就可以通过扫描索引来查询数据。

举个例子，我通过下面这条语句，对 length(name) 的计算结果建立一个名为 idx_name_length 的索引。

```sql
alter table t_user add key idx_name_length ((length(name)));
```

然后再次查询这条语句，这时候就会走索引了。

3. 计算导致索引失效

​	例：`select * from t_user where id + 1 = 10;`

​	失效原因类似第2点。

4. 对索引隐式类型转换

   两个例子：

   ```sql
   -- 例1 phone在表中为varchar类型
   select * from t_user where phone = 1300000001;	-- 不走索引
   -- 例2 id在表中为int类型
   select * from t_user where id = "1";		-- 走索引
   ```



​	因为MySQL中在遇到字符串和数字比较的时候，会自动把字符串转为数字，然后再进行比较。（得出这个结论：执行`select “10” > 9`，若返回1则代表是转换为数字进行比较，若返回0则代码转换为字符串。）

​	即相当于：

```sql
select * from t_user where CAST(phone AS signed int) = 1300000001;

select * from t_user where id = CAST("1" AS signed int);
```

故索引失效原因同上述第3点。

5. 联合索引非最左匹配

   对主键字段建立的索引叫做聚簇索引，对普通字段建立的索引叫做二级索引。

   那么**多个普通字段组合在一起创建的索引就叫做联合索引**，也叫组合索引。

   创建联合索引时，我们需要注意创建时的顺序问题，因为联合索引 (a, b, c) 和 (c, b, a) 在使用的时候会存在差别。

   联合索引要能正确使用需要遵循**最左匹配原则**，也就是按照最左优先的方式进行索引的匹配。

   比如，如果创建了一个 `(a, b, c)` 联合索引，如果查询条件是以下这几种，就可以匹配上联合索引：

   - where a=1；
   - where a=1 and b=2 and c=3；
   - where a=1 and b=2；

   需要注意的是，因为有查询优化器，所以 a 字段在 where 子句的顺序并不重要。

   但是，如果查询条件是以下这几种，因为不符合最左匹配原则，所以就无法匹配上联合索引，联合索引就会失效:

   - where b=2；
   - where c=3；
   - where b=2 and c=3；

6. where子句中使用or

   在 WHERE 子句中，如果在 OR 前的条件列是索引列，而在 OR 后的条件列不是索引列，那么索引会失效。

   例：id 是主键，age 是普通列

   ```sql
   select * from t_user where id = 1 or age = 18;
   ```

因为 OR 的含义就是两个只要满足一个即可，因此只有一个条件列是索引列是没有意义的，只要有条件列不是索引列，就会进行全表扫描。

若要使走索引，则需要将age字段设置为索引。

7. 索引字段使用is not null导致失效





## 网络相关

### 请求头里面有哪些信息

