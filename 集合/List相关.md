# List 相关

## AbstractList

`AbstractList` 是一个抽象类，它继承于 `AbstractCollection`。`AbstractList` 实现了 `List `接口中除 `size()`、`get(int location) `之外的函数。

## AbstractSequentialList

`AbstractSequentialList `是一个抽象类，它继承于 `AbstractList`。`AbstractSequentialList `实现了链表中，根据 index 索引值操作链表的全部函数。

## Vector 和 Stack

`Vector` 和 `Stack` 的设计目标是作为线程安全的 `List` 实现，替代 `ArrayList`。

- `Vector `- `Vector` 和 `ArrayList` 类似，也实现了 `List` 接口。但是， `Vector` 中的主要方法都是 `synchronized` 方法，即通过互斥同步方式保证操作的线程安全。
- `Stack` - `Stack `也是一个同步容器，它的方法也用 `synchronized` 进行了同步，它实际上是继承于 Vector 类。



## ArrayList🦁

### ArrayList 要点

ArrayList 是一个数组队列，相当于**动态数组**。**ArrayList** **默认初始容量大小为** **10** **，添加元素时，如果发现容量已满，会自动扩容为原始大小的 1.5 倍**。因此，应该尽量在初始化 ArrayList 时，为其指定合适的初始化容量大小，减少扩容操作产生的性能开销。

ArrayList 定义：

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

从 ArrayList 的定义，不难看出 ArrayList 的一些基本特性：

- ArrayList 实现了 List 接口，并继承了 AbstractList，它支持所有 List 的操作。
- ArrayList 实现了 RandomAccess 接口，支持随机访问。RandomAccess 是 Java 中用来被 List 实现，为 List 提供快速访问功能的。在 ArrayList 中，我们即可以通过元素的序号快速获取元素对象；这就是快速随机访问。
- ArrayList 实现了 Cloneable 接口，支持深拷贝。
- ArrayList 实现了 Serializable 接口，支持序列化，能通过序列化方式传输。
- ArrayList 是非线程安全的。

### ArrayList 原理

#### ArrayList 数据结构

 

```java
transient Object[] elementData;
private int size;
```

- elementData - 是一个 Object 数组，用于保存添加到 ArrayList 中的元素。

- size - 是动态数组的实际大小。默认初始容量（DEFAULT_CAPACITY）大小为 10 （可以在构造方法中指定初始大小），添加元素时如果发现容量已满，会自动扩容为原来1.5倍。

#### ArrayList 的序列化

ArrayList 具有动态扩容特性，因此保存元素的数组不一定都会被使用，那么就没必要全部进行序列化。为此，ArrayList 定制了其序列化方式。具体做法是：

- 存储元素的 Object 数组（即 elementData）使用 transient 修饰，使得它可以被 Java 序列化所忽略。
- ArrayList 重写了 writeObject() 和 readObject() 来控制序列化数组中有元素填充那部分内容。

#### ArrayList 的访问元素

```java
// 获取第 index 个元素
public E get(int index) {
    rangeCheck(index);
    return elementData(index);
}

E elementData(int index) {
    return (E) elementData[index];
}

private void rangeCheck(int index) {
	if (index >= size)
		throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

#### ArrayList 的添加元素🐧

添加元素时，如果发现容量已满，会自动扩容为原始大小的 1.5 倍。

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

扩容操作实际上是**调用** **Arrays.copyOf()** **把原数组拷贝为一个新数组**，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。

#### ArrayList 的删除元素

```java
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```

`ArrayList` 删除元素时，实际上是调用 `System.arraycopy()` 将 `index+1` 后面的元素都复制到 `index` 位置上，复制的代价很高。

#### ArrayList 的 Fail-Fast

`modCount` 用来记录 `ArrayList`结构发生变化的次数。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。

在进行序列化或者迭代等操作时，需要比较操作前后 `modCount` 是否改变，如果改变了需要抛出 `ConcurrentModificationException`。

```java
private void writeObject(java.io.ObjectOutputStream s) throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```



## LinkedList🐯

LinkedList 从数据结构角度来看，可以视为双链表。

### LinkedList 要点

`LinkedList` 基于双链表结构实现。由于是双链表，所以**顺序访问会非常高效，而随机访问效率比较低。**

`LinkedList` 定义：

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

从 `LinkedList` 的定义，可以得出 `LinkedList` 的一些基本特性：

- `LinkedList` 实现了 `List` 接口，并继承了 `AbstractSequentialList` ，它支持所有 `List` 的操作。
- `LinkedList` 实现了 `Deque` 接口，也可以被当作队列（`Queue`）或双端队列（`Deque`）进行操作，此外，也可以用来实现栈。
- `LinkedList` 实现了 `Cloneable` 接口，默认为**浅拷贝**。
- `LinkedList` 实现了 `Serializable` 接口，**支持序列化**。
- `LinkedList` 是**非线程安全**的。

### LinkedList 原理

#### LinkedList 的数据结构

`LinkedList` **内部维护了一个双链表**。

`LinkedList` 通过 `Node` 类型的头尾指针（`first` 和 `last`）来访问数据。

```java
// 链表长度
transient int size = 0;
// 链表头节点
transient Node<E> first;
// 链表尾节点
transient Node<E> last;
```

- `size` - **表示双链表中节点的个数，初始为 0**。
- `first` 和 `last` - **分别是双链表的头节点和尾节点**。

`Node` 是 `LinkedList` 的内部类，它表示链表中的元素实例。Node 中包含三个元素：

- `prev` 是该节点的上一个节点；
- `next` 是该节点的下一个节点；
- `item` 是该节点所包含的值。

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
    ...
}
```

#### LinkedList 的序列化

`LinkedList` 与 `ArrayList` 一样也定制了自身的序列化方式。具体做法是：

- 将 `size` （双向链表容量大小）、`first` 和`last` （双向链表的头尾节点）修饰为 `transient`，使得它们可以被 Java 序列化所忽略。
- 重写了 `writeObject()` 和 `readObject()` 来控制序列化时，只处理双向链表中能被头节点链式引用的节点元素。

#### LinkedList 访问元素🌵

`LinkedList` 访问元素的实现主要基于以下关键性源码：

```java
public E get(int index) {
	checkElementIndex(index);
	return node(index).item;
}

Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

获取 `LinkedList` 第 index 个元素的算法是：

- 判断 index 在链表前半部分，还是后半部分。
- 如果是前半部分，从头节点开始查找；如果是后半部分，从尾结点开始查找。

`LinkedList` 这种访问元素的性能是 `O(N)` 级别的（极端情况下，扫描 N/2 个元素）；相比于 `ArrayList` 的 `O(1)`，显然要慢不少。

**推荐使用迭代器遍历** `LinkedLis` **，不要使用传统的** `for` **循环**。注：foreach 语法会被编译器转换成迭代器遍历，但是它的遍历过程中不允许修改 `List` 长度，即不能进行增删操作。

#### LinkedList 添加元素

`LinkedList` 有多种添加元素方法：

- `add(E e)`：默认添加元素方法（插入尾部）
- `add(int index, E element)`：添加元素到任意位置
- `addFirst(E e)`：在头部添加元素
- `addLast(E e)`：在尾部添加元素

```java
public boolean add(E e) {
	linkLast(e);
	return true;
}

public void add(int index, E element) {
	checkPositionIndex(index);

	if (index == size)
		linkLast(element);
	else
		linkBefore(element, node(index));
}

public void addFirst(E e) {
	linkFirst(e);
}

public void addLast(E e) {
	linkLast(e);
}
```

LinkedList 添加元素的实现主要基于以下关键性源码：

```java
private void linkFirst(E e) {
	final Node<E> f = first;
	final Node<E> newNode = new Node<>(null, e, f);
	first = newNode;
	if (f == null)
		last = newNode;
	else
		f.prev = newNode;
	size++;
	modCount++;
}

void linkLast(E e) {
	final Node<E> l = last;
	final Node<E> newNode = new Node<>(l, e, null);
	last = newNode;
	if (l == null)
		first = newNode;
	else
		l.next = newNode;
	size++;
	modCount++;
}

void linkBefore(E e, Node<E> succ) {
	// assert succ != null;
	final Node<E> pred = succ.prev;
	final Node<E> newNode = new Node<>(pred, e, succ);
	succ.prev = newNode;
	if (pred == null)
		first = newNode;
	else
		pred.next = newNode;
	size++;
	modCount++;
}
```

算法如下：

- 将新添加的数据包装为 `Node`；
- 如果往头部添加元素，将头指针 `first` 指向新的 `Node`，之前的 `first` 对象的 `prev` 指向新的 `Node`。
- 如果是向尾部添加元素，则将尾指针 `last` 指向新的 `Node`，之前的 `last` 对象的 `next` 指向新的 `Node`。

#### LinkedList 删除元素

`LinkedList` 删除元素的实现主要基于以下关键性源码：

```java
public boolean remove(Object o) {
    if (o == null) {
        // 遍历找到要删除的元素节点
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        // 遍历找到要删除的元素节点
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}

E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```

算法说明：

- 遍历找到要删除的元素节点，然后调用 `unlink` 方法删除节点；
- `unlink` 删除节点的方法： 

- 如果当前节点有前驱节点，则让前驱节点指向当前节点的下一个节点；否则，让双链表头指针指向下一个节点。
- 如果当前节点有后继节点，则让后继节点指向当前节点的前一个节点；否则，让双链表尾指针指向上一个节点。

#### LinkedList 队列/栈方法

- 入队`offer()`
- 出队 `poll()`
- 获取队列第一个元素 `peek()`
- 入栈 `push()`
- 出栈 `pop()`



## List 常见问题

### Arrays.asList 问题点

1. **不能直接使用** `Arrays.asList` **来转换基本类型数组**

```java
int[] arr = { 1, 2, 3 };
List list = Arrays.asList(arr);

System.out.println(list);
System.out.println(list.size());
System.out.println(list.get(0).getClass());

// 输出结果
[[I@1b6d3586]
1
class [I
```

其原因是：`Arrays.asList` 方法传入的是一个泛型 T 类型可变参数，最终 `int` 数组整体作为了一个对象成为了泛型类型



2. `Arrays.asList` **返回的** `List` **不支持增删操作**

`Arrays.asList` 返回的 List 并不是我们期望的 `java.util.ArrayList`，而是 `Arrays` 的内部类 `ArrayList`。

查看源码，我们可以发现 `Arrays.asList` 返回的 `ArrayList` 继承了 `AbstractList`，但是并没有覆写 `add` 和 `remove` 方法。

3. **对原始数组的修改会影响到**`Arrays.asList`**生成的List**

解决方法很简单，重新 `new` 一个 `ArrayList` 初始化 `Arrays.asList` 返回的 `List` 即可：

```java
String[] arr = { "1", "2", "3" };
List list = new ArrayList(Arrays.asList(arr));
arr[1] = "4";
try {
    list.add("5");
} catch (Exception ex) {
    ex.printStackTrace();
}
log.info("arr:{} list:{}", Arrays.toString(arr), list);
```

NEW: 初始状态，线程被创建出来但没有被调用 `start()` 。

RUNNABLE: 运行状态，线程被调用了 `start()`等待运行的状态。

BLOCKED：阻塞状态，需要等待锁释放。

WAITING：等待状态，表示该线程需要等待其他线程做出一些特定动作（通知或中断）。

TIME_WAITING：超时等待状态，可以在指定的时间后自行返回而不是像 WAITING 那样一直等待。

TERMINATED：终止状态，表示该线程已经运行完毕。

------

著作权归JavaGuide(javaguide.cn)所有 基于MIT协议 原文链接：https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html

