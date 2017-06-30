---
layout: post
title: HashMap详解 for Jdk1.7
date: 2017-06-30 
tag: Java基础
---

### 小结 ###
	HashMap的初始容量为16,加载因子为0.75.
	
	   加载因子越大,填满的元素越多,好处是空间利用率高了,但是冲突的机会加大了,链表长度可能越来越长,查询效率降低.
	   加载因子越小,填满的元素越少,好处是冲突的机会减小了,但是空间浪费多了,表中的数据过于稀疏,很多空间还没用就开始扩容了.

	   如果机器内存足够,并且想要提高查询速度的话可以将加载因子设置小一点;相反,如果机器内存紧张,并且对查询速度没有什么要求的话
	   可以将加载因子设置大一些,但是查询速度会有所降低.

	HashMap其实就是一个Entry数组,Entry对象中又包含key和value,其中next也是一个Entry对象,形成链表,处理hash冲突.

	put():根据key算出hashCode值,决定此Entry的存储位置,如果两个Entry他们key的hash值相同,则他们的存储位置相同,
		  此时会再去通过equals方法去判断key是否相等,
		  如果相等,则此时新加入的Entry的value覆盖原先的Entry的value值,但key不会覆盖;
	 	  如果不相等,此时新加入的Entry会放在链头,与原先的Entry形成链表.
	
	get():根据key算出hashCode值,找到存储位置,再通过key的equals方法从链表中找到这个Entry对应的value值.

### 一.HashMap概述 ###
HashMap基于哈希表的 Map 接口的实现。底层使用hash算法+数组+链表的数据结构实现,key允许存储一个null值.

### 二.HashMap的数据结构 ###
HashMap的底层主要是基于数组和链表来实现的，它之所以有相当快的查询速度主要是因为它是通过计算散列码来决定存储的位置。HashMap中主要是通过key的hashCode来计算hash值的，只要hashCode相同，计算出来的hash值就一样。如果存储的对象对多了，就有可能不同的对象所算出来的hash值是相同的，这就出现了所谓的hash冲突。学过数据结构的同学都知道，解决hash冲突的方法有很多，HashMap底层是通过链表来解决hash冲突的。

![](http://i.imgur.com/k3go3q7.jpg)

 图中，紫色部分即代表哈希表，也称为哈希数组，数组的每个元素都是一个单链表的头节点，链表是用来解决冲突的，如果不同的key映射到了数组的同一位置处，就将其放入单链表中。
我们看看HashMap中Entry类的代码：
    
    /** Entry是单向链表。
      * 它是 “HashMap链式存储法”对应的链表。
      * 它实现了Map.Entry 接口，即实现getKey(), getValue(), setValue(V value), equals(Object o), hashCode()这些函数  
    **/  
    static class Entry<K,V> implements Map.Entry<K,V> {    
        final K key;    
        V value;    
        // 指向下一个节点    
        Entry<K,V> next;    
        final int hash;    
   
        // 构造函数。    
        // 输入参数包括"哈希值(h)", "键(k)", "值(v)", "下一节点(n)"    
        Entry(int h, K k, V v, Entry<K,V> n) {    
            value = v;    
            next = n;    
            key = k;    
            hash = h;    
        }    
   
        public final K getKey() {    
            return key;    
        }    
   
        public final V getValue() {    
            return value;    
        }    
   
        public final V setValue(V newValue) {    
            V oldValue = value;    
            value = newValue;    
            return oldValue;    
        }    
   
        // 判断两个Entry是否相等    
        // 若两个Entry的“key”和“value”都相等，则返回true。    
        // 否则，返回false    
        public final boolean equals(Object o) {    
            if (!(o instanceof Map.Entry))    
                return false;    
            Map.Entry e = (Map.Entry)o;    
            Object k1 = getKey();    
            Object k2 = e.getKey();    
            if (k1 == k2 || (k1 != null && k1.equals(k2))) {    
                Object v1 = getValue();    
                Object v2 = e.getValue();    
                if (v1 == v2 || (v1 != null && v1.equals(v2)))    
                    return true;    
            }    
            return false;    
        }    
   
        // 实现hashCode()    
        public final int hashCode() {    
            return (key==null   ? 0 : key.hashCode()) ^    
                   (value==null ? 0 : value.hashCode());    
        }    
   
        public final String toString() {    
            return getKey() + "=" + getValue();    
        }    
   
        // 当向HashMap中添加元素时，绘调用recordAccess()。    
        // 这里不做任何处理    
        void recordAccess(HashMap<K,V> m) {    
        }    
   
        // 当从HashMap中删除元素时，绘调用recordRemoval()。    
        // 这里不做任何处理    
        void recordRemoval(HashMap<K,V> m) {    
        }    
    }

HashMap其实就是一个Entry数组，Entry对象中包含了键和值，其中next也是一个Entry对象，它就是用来处理hash冲突的，形成一个链表。
 

### 三.HashMap源码分析 ###
   **1、关键属性**

　　先看看HashMap类中的一些关键属性：

    transient Entry[] table;//存储元素的实体数组
      
    transient int size;//存放元素的个数
      
    int threshold; //临界值   当实际大小超过临界值时，会进行扩容threshold = 加载因子*容量
    
    final float loadFactor; //加载因子
      
    transient int modCount;//被修改的次数

其中loadFactor加载因子是表示Hsah表中元素的填满的程度.

若:加载因子越大,填满的元素越多,好处是,空间利用率高了,但:冲突的机会加大了.链表长度会越来越长,查找效率降低。

反之,加载因子越小,填满的元素越少,好处是:冲突的机会减小了,但:空间浪费多了.表中的数据将过于稀疏（很多空间还没用，就开始扩容了）
冲突的机会越大,则查找的成本越高.

因此,必须在 "冲突的机会"与"空间利用率"之间寻找一种平衡与折衷. 这种平衡与折衷本质上是数据结构中有名的"时-空"矛盾的平衡与折衷.

如果机器内存足够，并且想要提高查询速度的话可以将加载因子设置小一点；相反如果机器内存紧张，并且对查询速度没有什么要求的话可以将加载因子设置大一点。不过一般我们都不用去设置它，让它取默认值0.75就好了。

**2、构造方法**

下面看看HashMap的几个构造方法：

	public HashMap(int initialCapacity, float loadFactor) {
          //确保数字合法
          if (initialCapacity < 0)
              throw new IllegalArgumentException("Illegal initial capacity: " +
                                                initialCapacity);
          if (initialCapacity > MAXIMUM_CAPACITY)
              initialCapacity = MAXIMUM_CAPACITY;
          if (loadFactor <= 0 || Float.isNaN(loadFactor))
              throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
 
         // Find a power of 2 >= initialCapacity
         int capacity = 1;   //初始容量
         while (capacity < initialCapacity)   //确保容量为2的n次幂，使capacity为大于initialCapacity的最小的2的n次幂
             capacity <<= 1;
 
         this.loadFactor = loadFactor;
         threshold = (int)(capacity * loadFactor);
         table = new Entry[capacity];
        init();
    }
 
     public HashMap(int initialCapacity) {
         this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
 
     public HashMap() {
         this.loadFactor = DEFAULT_LOAD_FACTOR;
         threshold = (int)(DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR);
         table = new Entry[DEFAULT_INITIAL_CAPACITY];
        init();
     }


我们可以看到在构造HashMap的时候如果我们指定了加载因子和初始容量的话就调用第一个构造方法，否则的话就是用默认的。默认初始容量为16，默认加载因子为0.75。我们可以看到上面代码中13-15行，这段代码的作用是确保容量为2的n次幂，使capacity为大于initialCapacity的最小的2的n次幂，至于为什么要把容量设置为2的n次幂，我们等下再看。
 
重点分析下HashMap中用的最多的两个方法put和get

 **3、存储数据**

	public V put(K key, V value) {
     // 若“key为null”，则将该键值对添加到table[0]中。
         if (key == null) 
            return putForNullKey(value);
     // 若“key不为null”，则计算该key的哈希值，然后将其添加到该哈希值对应的链表中。
         int hash = hash(key.hashCode());
     //搜索指定hash值在对应table中的索引
         int i = indexFor(hash, table.length);
     // 循环遍历Entry数组,若“该key”对应的键值对已经存在，则用新的value取代旧的value。然后退出！
         for (Entry<K,V> e = table[i]; e != null; e = e.next) { 
             Object k;
              if (e.hash == hash && ((k = e.key) == key || key.equals(k))) { //如果key相同则覆盖并返回旧值
                  V oldValue = e.value;
                 e.value = value;
                 e.recordAccess(this);
                 return oldValue;
              }
         }

     //修改次数+1
     modCount++;
     //将key-value添加到table[i]处
     addEntry(hash, key, value, i);
     return null;
	}

根据源码分析,首先会去判断key是否为null,如果为null,则将对象存储在数组中索引为0的位置,即table[0].

如果不为null,则根据key计算key的hash值,然后搜索该hash值在对应的table中的索引,如果计算出的hash值对应的索引的位置上有值了,则去判断key是否相等,如果相等,则用新的value值取代旧的value值,如果key不相等,则将新添加的元素,添加到链表头,next指向之前的元素.

	 //计算hash值的方法 通过键的hashCode来计算
     static int hash(int h) {
         // This function ensures that hashCodes that differ only by
         // constant multiples at each bit position have a bounded
         // number of collisions (approximately 8 at default load factor).
         h ^= (h >>> 20) ^ (h >>> 12);
         return h ^ (h >>> 7) ^ (h >>> 4);
     }


得到hash码之后就会通过hash码去计算出应该存储在数组中的索引，计算索引的函数如下：
 
     static int indexFor(int h, int length) { //根据hash值和数组长度算出索引值
         return h & (length-1);  //这里不能随便算取，用hash&(length-1)是有原因的，这样可以确保算出来的索引是在数组大小范围内，不会超出
     }

这个我们要重点说下，我们一般对哈希表的散列很自然地会想到用hash值对length取模（即除法散列法），Hashtable中也是这样实现的，这种方法基本能保证元素在哈希表中散列的比较均匀，但取模会用到除法运算，效率很低，HashMap中则通过h&(length-1)的方法来代替取模，同样实现了均匀的散列，但效率要高很多，这也是HashMap对Hashtable的一个改进。

 
接下来，我们分析下为什么哈希表的容量一定要是2的整数次幂。首先，length为2的整数次幂的话，h&(length-1)就相当于对length取模，这样便保证了散列的均匀，同时也提升了效率；其次，length为2的整数次幂的话，为偶数，这样length-1为奇数，奇数的最后一位是1，这样便保证了h&(length-1)的最后一位可能为0，也可能为1（这取决于h的值），即与后的结果可能为偶数，也可能为奇数，这样便可以保证散列的均匀性，而如果length为奇数的话，很明显length-1为偶数，它的最后一位是0，这样h&(length-1)的最后一位肯定为0，即只能为偶数，这样任何hash值都只会被散列到数组的偶数下标位置上，这便浪费了近一半的空间，因此，length取2的整数次幂，是为了使不同hash值发生碰撞的概率较小，这样就能使元素在哈希表中均匀地散列。

　这看上去很简单，其实比较有玄机的，我们举个例子来说明：
　　假设数组长度分别为15和16，优化后的hash码分别为8和9，那么&运算后的结果如下： 

       h & (table.length-1)                     hash                             table.length-1
       8 & (15-1)：                                 0100                   &              1110                   =                0100
       9 & (15-1)：                                 0101                   &              1110                   =                0100
       -----------------------------------------------------------------------------------------------------------------------
       8 & (16-1)：                                 0100                   &              1111                   =                0100
       9 & (16-1)：                                 0101                   &              1111                   =                0101
 

从上面的例子中可以看出：当它们和15-1（1110）“与”的时候，产生了相同的结果，也就是说它们会定位到数组中的同一个位置上去，这就产生了碰撞，8和9会被放到数组中的同一个位置上形成链表，那么查询的时候就需要遍历这个链 表，得到8或者9，这样就降低了查询的效率。同时，我们也可以发现，当数组长度为15的时候，hash值会与15-1（1110）进行“与”，那么 最后一位永远是0，而0001，0011，0101，1001，1011，0111，1101这几个位置永远都不能存放元素了，空间浪费相当大，更糟的是这种情况中，数组可以使用的位置比数组长度小了很多，这意味着进一步增加了碰撞的几率，减慢了查询的效率！而当数组长度为16时，即为2的n次方时，2n-1得到的二进制数的每个位上的值都为1，这使得在低位上&时，得到的和原hash的低位相同，加之hash(int h)方法对key的hashCode的进一步优化，加入了高位计算，就使得只有相同的hash值的两个值才会被放到数组中的同一个位置上形成链表。

　　 所以说，当数组长度为2的n次幂的时候，不同的key算得得index相同的几率较小，那么数据在数组上分布就比较均匀，也就是说碰撞的几率小，相对的，查询的时候就不用遍历某个位置上的链表，这样查询效率也就较高了。


**根据上面 put 方法的源代码可以看出，当程序试图将一个key-value对放入HashMap中时，程序首先根据该 key 的 hashCode() 返回值决定该 Entry 的存储位置：如果两个 Entry 的 key 的 hashCode() 返回值相同，那它们的存储位置相同。如果这两个 Entry 的 key 通过 equals 比较返回 true，新添加 Entry 的 value 将覆盖集合中原有 Entry 的 value，但key不会覆盖。如果这两个 Entry 的 key 通过 equals 比较返回 false，新添加的 Entry 将与集合中原有 Entry 形成 Entry 链，而且新添加的 Entry 位于 Entry 链的头部**

具体说明继续看 addEntry() 方法的说明。

    void addEntry(int hash, K key, V value, int bucketIndex) {
         Entry<K,V> e = table[bucketIndex]; //如果要加入的位置有值，将该位置原先的值设置为新entry的next,也就是新entry链表的下一个节点
         table[bucketIndex] = new Entry<>(hash, key, value, e);
         if (size++ >= threshold) //如果大于临界值就扩容
             resize(2 * table.length); //以2的倍数扩容
     }
　　　

参数bucketIndex就是indexFor函数计算出来的索引值，第2行代码是取得数组中索引为bucketIndex的Entry对象，第3行就是用hash、key、value构建一个新的Entry对象放到索引为bucketIndex的位置，并且将该位置原先的对象设置为新对象的next构成链表

第4行和第5行就是判断put后size是否达到了临界值threshold，如果达到了临界值就要进行扩容，HashMap扩容是扩为原来的两倍。


**4、调整大小**

resize()方法如下：
 重新调整HashMap的大小，newCapacity是调整后的单位

	void resize(int newCapacity) {
          Entry[] oldTable = table;
          int oldCapacity = oldTable.length;
          if (oldCapacity == MAXIMUM_CAPACITY) {
              threshold = Integer.MAX_VALUE;
              return;
         }
  
          Entry[] newTable = new Entry[newCapacity];
         transfer(newTable);//用来将原先table的元素全部移到newTable里面
         table = newTable;  //再将newTable赋值给table
         threshold = (int)(newCapacity * loadFactor);//重新计算临界值
     }

新建了一个HashMap的底层数组，上面代码中第10行为调用transfer方法，将HashMap的全部元素添加到新的HashMap中,并重新计算元素在新的数组中的索引位置
 
 
当HashMap中的元素越来越多的时候，hash冲突的几率也就越来越高，因为数组的长度是固定的。所以为了提高查询的效率，就要对HashMap的数组进行扩容，数组扩容这个操作也会出现在ArrayList中，这是一个常用的操作，而在HashMap数组扩容之后，最消耗性能的点就出现了：原数组中的数据必须重新计算其在新数组中的位置，并放进去，这就是resize。
 
   那么HashMap什么时候进行扩容呢？当HashMap中的元素个数超过数组大小/*loadFactor时，就会进行数组扩容，loadFactor的默认值为0.75，这是一个折中的取值。也就是说，默认情况下，数组大小为16，那么当HashMap中元素个数超过16\*0.75=12的时候，就把数组的大小扩展为 2\*16=32，即扩大一倍，然后重新计算每个元素在数组中的位置，扩容是需要进行数组复制的，复制数组是非常消耗性能的操作，所以如果我们已经预知HashMap中元素的个数，那么预设元素的个数能够有效的提高HashMap的性能。
 
** 5、数据读取**

	public V get(Object key) {   
	    if (key == null)   
	        return getForNullKey();   
	    int hash = hash(key.hashCode());   
	    for (Entry<K,V> e = table[indexFor(hash, table.length)];   
	        e != null;   
	        e = e.next) {   
	        Object k;   
	        if (e.hash == hash && ((k = e.key) == key || key.equals(k)))   
	            return e.value;   
	    }   
	    return null;   
	}  

从HashMap中get元素时，首先计算key的hashCode，找到数组中对应位置的某一元素，然后通过key的equals方法在对应位置的链表中找到需要的元素。