---
layout: post
title: Interview
date: 2017-07-03 
tag: 学习总结
---

## 一.基础 ##

### 1.Java的数据类型 ###

![](http://i.imgur.com/fyd9tyN.png)

常量分类:数字型,字符型,字符串型,布尔型,null型.

### 2.数据类型转换 ###

![](http://i.imgur.com/zjXuB9E.png)

### 3.选择排序 ###

![](http://i.imgur.com/yI9veOW.png)

 	@Test
	public void xuanzheSort() {
		int[] arr = { 12, 3, 144, -88, 7, 3, 9 };

		for (int i = 0; i < arr.length - 1; i++) {
			for (int j = i + 1; j < arr.length; j++) {
				if(arr[i] > arr[j]){
					int temp = arr[i];
					arr[i] = arr[j];
					arr[j] = temp;
				}
			}
		}
		
		for (int i = 0; i < arr.length; i++) {
			System.out.print(arr[i] + " ");
		}

	}


### 4.冒泡排序 ###

	@Test
	public void maopaoSort() {
		int[] arr = { 12, 3, 144, -88, 7, 3, 9 };

		for (int i = 0; i < arr.length - 1; i++) {
			for (int j = 0; j < arr.length - i - 1; j++) {
				if (arr[j] > arr[j + 1]) {
					int temp = arr[j];
					arr[j] = arr[j + 1];
					arr[j + 1] = temp;
				}
			}
		}

		for (int i = 0; i < arr.length; i++) {
			System.out.print(arr[i] + " ");
		}

	}


### 5.成员变量与局部变量的区别 ###

**1，从定义上讲**

成员变量定义在类中，属于对象的

局部变量定义函数中，属于函数的。（超函数外就无法访问）

**2，从存储位置上讲（内存位置）**

成员变量随着对象的创建，在堆中空间存储

局部变量随着函数的调用，会存储在栈空间下

**3，从存活时间(生命周期)上讲**

成员变量的存活时间是随着对象的创建而存在，随着对象的销毁而消失，来计算时间的

局部变量的存活时间是随着函数的入栈而存在，随着函数的弹栈而消失，来计算时间

**4，从使用上讲**

成员变量必须是通过对象来访问（没有对象不能直接访问）

局部变量可以直接在当前函数中内访问

**5，从初始值上讲**

成员变量都具备默认初始值

局部变量没有默认初始值，必须手动的给变量初始化


### 6.类的加载过程 ###

![](http://i.imgur.com/B3vAVMN.png)

类的加载过程：

1，当使用java命令运行java程序时，此时JVM启动，并去方法区下找java命令后面跟的类是否存在，如果不存在，则把类加载到方法区下

2，在类加载到方法区时，会分为两部分：先加载非静态内容到方法区下的非静态区域内，再加载静态内容到方法区下的静态区域内

3，当非静态内容载完成之后，就会加载所有的静态内容到方法区下的静态区域内

	3.1  先把所有的静态内容加载到静态区域下

	3.2  所有静态内容加载完之后，对所有的静态成员变量进行默认初始化

	3.3  当所有的静态成员变量默认初始化完成之后，再对所有的静态成员变量显式初始化

4，当所有的静态成员变量显式初始化完成之后，JVM自动执行静态代码块（静态代码块在栈中执行）[如果有多个静态代码，执行的顺序是按照代码书写的先后顺序执行]

5，所有的静态代码块执行完成之后，此时类的加载完成


### 7.对象的创建过程 ###

对象的创建过程：

1，当在main方法中创建对象时，JVM会先去方法区下找有没有所创建对象的类存在，有就可以创建对象了，没有则把该类加载到方法区

2，在创建类的对象时，首先会先去堆内存中开辟一块空间，开辟完后分配该空间(指定地址)

3，当空间分配完后，加载对象中所有的非静态成员变量到该空间下

4，所有的非静态成员变量加载完成之后，对所有的非静态成员进行默认初始化

5，所有的非静态成员默认初始化完成之后，调用相应的构造方法到栈中

6，在栈中执行构造函数时，先执行隐式三步，再执行构造方法中书写的代码

	6.1  构造方法中的隐式三步：
	
			第一步：执行super()语句      调用父类的没有参数的构造方法
			
			第二步：对所有的非静态成员变量进行显式初始化（在定义成员变量时后面有赋值）
			
			第三步：所有的非静态成员变量显式初始化完成之后，执行构造代码块
	
	6.2 只有隐式三步执行完后，才会执行构造方法中书写的代码

7，当整个构造方法全部执行完，此对象创建完成，并把堆内存中分配的空间地址赋给对象名（此时对象名就指向了该空间）


### 8.单例设计模式 ###

	// 懒汉式单例模式
	// 1.私有化构造方法
	// 2.创建一个属于本类的对象
	// 3.对外提供一个可以访问创建出来对象的方法
	public class Single {
	
		private Single(){
			
		}
		
		private static Single instance = null;
		
		public static Single getInstance(){
			if(instance == null){
				instance = new Single();
			}
			return instance;
		}
		
		public void test(){
			System.out.println("测试");
		}
	}


### 9.集合 ###

	Collection接口:
  		 向集合中添加元素  add(Object obj)    addAll(Collection coll)
 		 从集合中删除元素  remove(Object obj)   removeAll(Collection coll)
  		 Collection集合中没有修改集合中元素的方法（Collection子类中有）
  		 Collection中没有直接获取元素的方法，只有一个遍历集合的方法iterator
  		 contains()判断集合中是否包含指定的元素
  		 clear() 删除集合中所有的元素

 	 |-----List接口：有序、可以存储重复元素、可以通过角标来操作集合(添、删、改)List集合有一个专属的迭代器：ListIterator

      通过角标来添加元素   add(int index,Object obj)
      通过角标来删除元素   remove(int index)  返回所删除的元素
      通过角标来修改元素   set(int index, Object obj) 返回修改之前的元素
      通过角标来获取元素   get(int index)

      	|----ArrayList：实现了List接口，底层是可变数组结构
                   查询速度快，增删速度慢
     	 |---LinkedList：实现了List接口，底层使用链表结构(有头有尾)，特有方法都是围绕着头和尾去设计的
                   查询速度慢，增删比较快

	          addFirst(Object  obj)  在头角标添加元素
	          addLast(Object  obj)  在尾角标添加元素
	          removeFirst()  删除头角标元素
	          removeLast()  删除尾角标元素
	          getFirst()  获取头角标元素
	          getLast()  获取尾角标元素
      |----Vector   底层使用数组结构

  	|-----Set接口：无序、不能存储重复元素，没有角标只能使用迭代器来遍历集合
               Set接口是继承了Collection接口，所以Set集合中的方法都来自于Collection
      |----HashSet：实现了Set接口，底层是哈希表结构，只能使用Iterator来遍历
          |----LinkedHashSet类：底层是哈希+链表结构
      |----TreeSet：实现了Set接口，底层是二叉树结构，在添加元素时会对元素进行排序
                  排序方式有两种：
					1，自然排序 （前提：对象必须具备compareTo方法[实现Comparable接口]）
					2，比较器排序 （使用Comparetor进行排序）
	                注意：当对象同时具有自然排序和比较器排序，优先比较器排序

	Map接口：不能直接使用迭代器遍历。
  		添加：  put(K key,V value)    key在集合中是唯一的   value可以存储重复元素
	        put方法在添加对象时，key就表示键，value就表示值，put存储的是一对键值对
	        当存储对象时，会按key的名称先进行查找，没有找到相同的key，就存储value对象，并返回null
	        如果存储时有相同的key已经在map集合中存在了，value对象会覆盖之前的对象，并返回之前的对象(旧的value对象)

		删除：  remove(Object key)
	        根据指定的key值去map集合中查找相同的key，如果没有相同的key，不执行删除操作，并返回null
	        如果存在相同的key值，就删除相同key值对应value的一对对象(key-value同时删除) ，并返回被删除的value(旧的value对象)

		修改：  使用put方法进行修改

		获取：  get(Object key)
	        根据指定的key去集合中查找相同的key，如果有相同的key，就返回查找到的key所对应的value对象，如果没有相同的key，返回null
	        size() 获取map集合所存储key-value对的个数

		判断：  containsKey(Object key)  根据指定的key去map集合判断有没有相同的key存在
	         containsValue(Object value) 根据指定的value去map集合判断有没有相同的value
	         isEmpty()  判断map集合是否为空

 	 |-----Hashtable类：数据结构：哈希表； 不允许使用null作为key和value，已经被HashMap代替
         |----Properties类：属性集。存储键值都是字符串，可以利用流来对键值进行操作
                         后面学习到IO流时再讲解
  	|----HashMap类：数据结构：哈希表； 允许使用null作为key和value，但是key只能有一个null（key要保证唯一性），value可以存储重复对象
         |----LinkedHashMap类：链表+哈希表。可以保证存取的元素顺序是一致的
  	|----TreeMap类：数据结构：二叉树。可以按照存储的key进行排序



### 10.IO流 ###

	字节流：字节输入流、字节输出流
	   InputStream：所有字节输入流的超类
	      |----FileInputStream：用来对文件进行读取操作，按字节读取
	         构造方法：
	FileInputStream  fis = new FileInputStream(“文件路径”);
	FileInputStream  fis = new FileInputStream(文件对象);
	常用方法：
	close(); 关闭字节输入流对象
	int ch= read(); 一次读取一个字节数据
	int len = read(byte[]) 一次读取多个字节数据，存储在byte[]中
	
	   OutputStream：所有字节输出流的超类
	      |----FileOutputStream：用来对文件进行写入操作，按字节进行写入
	          构造方法：
	          FileOutputStream fos=new FileOutputStream(“文件路径”);
	FileOutputStream fos=new FileOutputStream(文件对象);
	FileOutputStream fos=new FileOutputStream(“文件路径”,续写方式);
	FileOutputStream fos=new FileOutputStream(文件对象,续写方式);
	          常用方法：
	           close();关闭字节输出流对象
	           write(byte)一次写入一个字节数据
	           wirte(byte[])一次写入多个字节数据
	           wirte(byte[],0,len)一次写入从byte数组中0角标开始到len个长度字节数据
	字节流中的高效流：缓冲区对象(本身不具备读写功能，需要借助具有读写功能的流)
	BufferedInputStream、BufferedOutputStream
	缓冲区对象内部维护了一个缓冲区数组，大小是1024*8
	高效流的特点：减少IO读取次数，可以提升对文件的读写效率
	缓冲区对象的构造方法：
	  BufferedInputStream  bis = new BufferedInputStream(new 字节输入流对象(“文件路径”));
	  BufferedOutputStream bos= new BufferedOutputStream(new 字节输出流对象(“文件”))
	
	字符流：字符输入流、字符输出流
	字符流 = 字节流 +　编码表
	  Reader：所有字符输入流的超类
	     |----InputStreamReader：把字节流转换为字符流的转换流，可以指定编码表
	          |----FileReader：用来读取字符文件的字符输入流。按字符来读取数据
	          构造方法：
	          FileReader fr=new FileReader(“文件路径”/文件对象);使用默认的编码表来读取
	          常用方法：
	          close();
	          int ch = read();一次读取一个字符
	          int len = read(char[])一次读取多个字符数据，存储在char[]中
	 Writer：所有字符输出流的超类
	     |----OutputStreamWriter：把字符流转为字节流的转换流可以指定编码表
	          |----FileWriter：用来向文件中写入字符数据。按照字符来写入
	          构造方法：
	           FileWriter  fw=new FileWriter(“文件路径”/文件对象); 默认的编码表
	           FileWriter  fw=new FileWriter(“文件路径”/文件对象,续写方式);
	          常用方法：
	           close();
	           write(char);  把字符数据转为字节数据，存储字节缓冲区中
	           write(char[],0,len)
	           write(String)
	           flush();　把存储在字节缓冲区中的数据，写入到文件中
	
	字符缓冲区对象：
	 BufferedReader 高效流：字符输入流，可以按行读取
	 BufferedWriter 高效流：字符输出流，可以写入一行，并使用newLine跨平台换行


### 11.线程 ###

![](http://i.imgur.com/83BVYBj.png)

**创建线程的两种方式:**

1.继承Thread线程类，并重写run()方法

2.实现Runnable接口，重写run()方法

**wait()和sleep()方法的区别：**

	wait方法是让线程处于等待状态，线程会一直处于等待状态，进到唤醒线程
	
	sleep方法也是让线程处于等待(睡眠)状态，时间一到，线程就是变为运行状态或临时阻塞
	
	wait方法可以设置时间，也可以不设置时间
	
	sleep方法必须设置时间、
	
	wait方法属于对象锁的；sleep方法属于线程的
	
	wait方法只能在同步代码块中使用；
	
	sleep方法可以在同步代码块中使用也可以在同步外部使用


### 12.TCP ###

![](http://i.imgur.com/qjW3oFC.png)

TCP协议：面向有连接、传输数据安全、传输速度比较慢、没有传输大小限定

TCP协议通信需要经过三次握手之后，可以实现数据的传输




### 1.聚合索引与非聚合索引 ###

**聚合索引:该索引中键值的逻辑顺序决定了表中数据的物理顺序.**

	create clustered index 索引名 on 表(字段)


**非聚合索引:数据存储在一个地方,索引存储在一个地方,索引带有指针指向数据的存储位置**

	create nonclustered index 索引名 on 表(字段)


### 2.行转列 ###

**concat(str1,str2) 将多个字符串连接成一个字符串**

**concat_ws(separator,str1,str2) 指定参数间的分隔符**

**group_concat() 可以使用distinct去重,可以使用order by排序.**


### 3.乐观锁和悲观锁 ###

**悲观锁原理:**使用数据库内部锁机制,进行table的锁定,在A修改数据时,A就将数据锁定,B此时无法进行修改.

**假设丢失更新会发生**

在mysql中有两种常用锁,读锁与写锁

**读锁(共享锁):**一张表可以添加多个读锁,如果表添加读锁,该表不可以修改.

**写锁(排他锁):**一张表只能添加一个排他锁,排他锁和其他共享锁和排他锁都互斥.

**乐观锁原理:**使用的不是数据库锁机制,而是一个特殊标记字段,判断控制字段的状态和内容,得知数据是否发生并发访问.

**假设丢失更新不会发生**

适用于写比较少的情况下


