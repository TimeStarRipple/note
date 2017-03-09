# 对象对象对象对象java集合

## 定义

对象的可变数组，是一种常用的数据结构，可以实现栈队列等，还有一些具有映射关系的关联数组

java集合类主要由两个接口派生而出：Collection，Map

他们石java集合框架的跟接口，两个接口包含了一些子接口以及实现类。

下面是Collection的继承树：

![](collection-1.png)

下面是Map的集合树：

![](/java/images/collection-2.png)

## Collection

### 常用方法：

查看当前数据个数：

int size\(\)：数据个数

boolean isEmpty\(\)：是否排空，空为true

新增：

boolean add\(Object O\)：添加数据，成功返回true

boolean addAll\(Collection c\)：添加数据，成功返回true

删除：

boolean remove\(Object O\)：删除数据O，如果有多个，只删除第一个，成功返回true

boolean removeAll\(Collection c\)：删除集合中包含在c中的所有数据，删除一个或一个以上成功返回true

boolean retainAll\(Collection c\)：删除非集合c中的数据，取两者交集，删除一个或一个以上成功返回true

boolean clear\(\)：清空集合

查找：

boolean contains\(Object O\)：是否包含对象

boolean containsAll\(Collection c\)：是否包含集合

转换：

Iterator iterator\(\)：转换成迭代器

Object\[\] toArray\(\)：把集合转化成一个数组

### 遍历的方法

for

```java
for (String str : list)
{
    System.out.println(str);
}
```

Iterator

```
Iterator<String> iterator = list.iterator();
while (iterator.hasNext())
{
    System.out.println(iterator.next());
}
```

foreach

```
list.forEach(str -> System.out.println(str));
```

所有遍历都不能在遍历的时候使用集合本身删除集合要不会报错

## Iterator

Iterator迭代器：采用快速失败机制，一旦在迭代过程中发现集合被修改，程序会CocurrentModificationException异常，避免其他线程对集合修改，而引发问题

```java
List<String> list = new ArrayList<>();
list.add("123");
list.add("234");
list.add("345");

Iterator<String> iterator = list.iterator();
while (iterator.hasNext())
{
    String str = iterator.next();
    if("234".equals(str))
    {
        iterator.remove();
    }
}

list.forEach(str -> System.out.println(str));
```

迭代器还是指向了原来数组的元素，你可以使用iterator删除他的值，但无法改变的值，因为他仅仅是值传递

## List

描述：有序，重复的集合

ArrayList  
Vector

--&gt;Stack  
LinkedList

数组转化List

Arrays工具类采用toList\(\)，可以将数组转化成固定的不可以删除和添加的List

性能间的比较：

## Set

描述：无序，不可重复的集合

常用的集合：

HashSet：快速找到值，可以为null  
--&gt;LinkedHashSet：记录插入顺序，查找性能略低，但是迭代访问有很好的性能  
SortedSet（接口）--&gt;TreeSet：可以排序的，默认自然排序，从小到大，采用红黑树的数据结构，请不要添加一个可变变量，后期变化但是它排序不会调整

EnumSet：以位向量进行存储，存储紧凑，占用内存小，效率高，不可以为null，没有构造方法，通过使用集合（值是同一个枚举类的）以及枚举类创建

#### 性能间的比较：

HashSet会比TreeSet性能好，因为TreeSet要额外用红黑树维护排序

HashSet在插入，删除，查找方面会比LinkedHashSet好，因为后者使用了链表，需要维护插入顺序，但是后者遍历方便

EnumSet效率高，但是限制大，值必须是同一个枚举类的

#### 对象比较：

实现Comparable接口，里面有一个compareTo的方法是比较方法，返回0为相等，返回整数是调用他的对象更大，负数是更小，请保证它的结果和equals相同，对于TreeSet来说，它会使用compareTo进行判定是否相等，相等是不允许添加的

#### 同步：

以上三种集合均不同步，要用工具类在创建时使用同步方法创建

```java
SortedSet<String> strings = Collections.synchronizedSortedSet(new TreeSet<String>());
```

## Map

描述：有映射关系的集合  
Hashtable  
HashMap  
--&gt;LinkedHashMap  
SortedMap（接口）--&gt;TreeMap

EnumMap

性能间的比较：

## Queue

队列

PriorityQueue：实现类，不是纯粹的队列，对插入的数据进行了排序  
Deque：接口，双端队列，可以用作栈

--&gt;ArrayDeque：

Collection是Set，Queue，List的父接口

## 问题

### 1.HashTable和HashMap区别

1. 继承不同。  
   public class Hashtable extends Dictionary implements Map public class HashMap extends AbstractMap implements Map

2. Hashtable中的方法是同步的，而HashMap中的方法在缺省情况下是非同步的。在多线程并发的环境下，可以直接使用Hashtable，但是要使用HashMap的话就要自己增加同步处理了。

3. Hashtable中，key和value都不允许出现null值。在HashMap中，null可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为null。当get\(\)方法返回null值时，即可以表示HashMap中没有该键，也可以表示该键所对应的值为null。因此，在HashMap中不能由get\(\)方法来判断HashMap中是否存在某个键，而应该用containsKey\(\)方法来判断。

4. 两个遍历方式的内部实现上不同。Hashtable、HashMap都使用了Iterator。而由于历史原因，Hashtable还使用了Enumeration的方式 。

5. 哈希值的使用不同，HashTable直接使用对象的hashCode。而HashMap重新计算hash值。

6. Hashtable和HashMap它们两个内部实现方式的数组的初始大小和扩容的方式。HashTable中hash数组默认大小是11，增加的方式是 old\*2+1。HashMap中hash数组的默认大小是16，而且一定是2的指数。

链接： [https:\/\/www.nowcoder.com\/questionTerminal\/c5e932bcec3a46cbb9976eea0783e555](https://www.nowcoder.com/questionTerminal/c5e932bcec3a46cbb9976eea0783e555)

来源：牛客网

