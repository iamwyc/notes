## Java 异常机制
Java把异常作为一种类，当做对象来处理。所有异常类的基类是Throwable类，两大子类分别是Error和Exception。  
这些异常类可以分为三种类型：系统错误、异常和运行时异常。  
* 系统错误  
Java虚拟机抛出，用Error类表示。Error类描述的是内部系统错误，例如Java虚拟机崩溃。这种情况仅凭程序自身是无法处理的，在程序中也不会对Error异常进行捕捉和抛出。  
* 运行时异常  
程序运行过程中出现错误，才会被检查的异常。例如：类型错误转换，数组下标访问越界，空指针异常、找不到指定类等等  
* 检查异常  
自于Exception且非运行时异常都是检查异常，编译器会强制检查并通过try-catch块来对其捕获，或者在方法头声明该异常，交给调用者处理.例如IO异常  
## HashMap缺点及内部实现
本质上是默认容量为16，加载因子为0.75的数组，每次扩容2倍。  Threshold=容量* 加载因子  
加载因子：当前大小==容量*加载因子时开始扩容  
但是每个Node又可以构成一个链表  
即同样的hash值在一个链表里，链表法解决hash冲突  

```
    transient Node<K,V>[] table;
	static class Node<K,V> implements Map.Entry<K,V> {
	        final int hash;
	        final K key;
	        V value;
	        Node<K,V> next;
	}

```
    
JDK1.8：加入了红黑树
即使负载因子和Hash算法设计的再合理，也免不了会出现拉链过长的情况，一旦出现拉链过长，则会严重影响HashMap的性能。于是，在JDK1.8版本中，对数据结构做了进一步的优化，引入了红黑树。
当链表长度超过8的时候，链表结构会改变成红黑树

-| HashMap |  HashTable  
-|-|-
null的键值(key)和值(value) | 可以 | 不可以 |
synchronized | 线程不安全 | 线程安全 |
速度 | 快 | 慢 |
在具体实现是新value插入到链表（JDK8） | 放在链表尾 | 放在链表头 |
红黑树（JDK8） | 使用 | 不使用 |

## ConcurrentHashMap内部实现
	JDK6、7、8的实现不一样 http://www.importnew.com/22007.html

## ArrayList与LinkedList的内部实现与区别
* ArrayList(线程不安全)
1. 本质是初始长度为10的数组。可以放null  

```
    transient Object[] elementData;
```

2. 删除obj  
	获取到对象的index。即：把index后的数组向前挪已位

```
	private void fastRemove(int index) {
	        modCount++;
	        int numMoved = size - index - 1;
	        if (numMoved > 0)
	            System.arraycopy(elementData, index+1, elementData, index,
	                             numMoved);
	        elementData[--size] = null; // clear to let GC do its work
	    }
```

3. 扩容

```
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

* LinkedList(线程不安全)
1. 本质是个双向链表

```
	transient Node<E> first;
	transient Node<E> last;
	private static class Node<E> {
	        E item;
	        Node<E> next;
	        Node<E> prev;
	}

```

## Java线程池

## Java lock的实现，公平锁、非公平锁

