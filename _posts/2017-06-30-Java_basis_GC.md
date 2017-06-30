---
layout: post
title: GC原理
date: 2017-06-30 
tag: Java基础
---

### 一.垃圾回收机制 ###

回收超出作用域或者引用计数为空的对象;从根节点开始搜索,搜索不到的对象,而且经过一次标记-清理仍然没有复活的对象的内存空间.
原理:把对象分为年青代(Young)、年老代(Tenured)、持久代(Perm)，对不同生命周期的对象使用不同的算法。(基于对对象生命周期分析)

![](http://i.imgur.com/1sNfqKC.png)![](http://i.imgur.com/S4Pia6e.png)


### 二.堆内存分配 ###

	年轻代(Young):
		年轻代的生命周期很短,一般新生成的对象会存放在该区的Eden区内,如果Eden区满时,还存活的对象将被复制到survivor(两个中的一个),当一个survivor满时,此区的存活对象将被复制到另外一个survivor区,当另外一个survivor也满时,从前一个survivor区复制过来并且此时还存活的对象,将可能被复制到年老代.

		年轻代又分为3个区
		Eden:对象创建被分配到这个区
		survivor1区:copying算法中的from space
		survivor2区:copying算法中的to space

		年轻代的GC称为Minor GC. 经过数次minor gc 之后依旧存活的对象,将被移出年轻代,移到年老代.

	年老代(Tenured):
		年老代的生命周期较长,在年轻代中经过多次minor gc之后依然存活的对象,就会被复制到年老代中.
		年老代中的GC成为Full GC

	持久代(Permanet):
		持久代比较稳定,主要用于存放classLoader信息,比如Class信息和method信息.


### 三.内存申请流程 ###
JVM会试图为相关的Java对象在年轻代的Eden区中初始化一块内存区域.
当Eden区空间足够时,内存申请结束,否则执行下一步. 执行Minor GC
JVM试图释放在Eden中所有不活跃的对象,释放后若Eden空间仍然不足以放入新对象,JVM则试图将Eden区中的活跃对象放入survivor区
当一个survivor满时,此区存活的对象将被复制到另外一个survivor区,当另外一个survivor也满时,从前一个survivor区复制过来并且此时还存活的对象将被移到年老代.
当年老代空间也不够时,JVM会在年老代进行完全的垃圾回收(Full GC).
Full GC后,若survivor区及年老代仍然无法存放从Eden区复制过来的对象时,则会导致JVM无法在Eden区为新对象申请内存空间,即出现"Out of Memory",内存溢出.

科普:
   内存溢出(out of memory):是指程序申请内存时,没有足够的内存空间供其使用,出现out of memory
   内存泄露(memory leak):是指程序在申请内存后,无法释放已申请的内存空间,导致这部分内存不可用的情况.
如何避免:
   尽早释放无用对象的引用.
   程序进行字符串处理时,尽量避免使用String,而应使用StringBuffer.
   尽量少使用静态变量 静态变量是全局的,GC不会回收
   避免集中创建对象.
   尽量运用对象池技术以提高系统性能;生命周期长的对象拥有生命周期短的对象时容易引发内存泄漏.
   不要在经常调用的方法中创建对象,尤其是循环中创建对象.可以适当的使用hashtable或者vector创建一组对象容器,然后从容器中取那些对象,而不是每次new之后再丢弃
   优化配置,见标题7

### 四.GC有两种类型 ###
1):**Minor GC:**当新对象生成,并且在Eden申请空间失败时触发.
	Minor GC 是对Eden区域进行GC,清除非存活对象,并且把尚且
	存活的对象移入到Survivor区.然后整理Survivor的两个区.
	这种方式的GC,是对Eden区进行,不会影响到年老代.
	因为大部分对象都是从Eden区开始的,同时Eden区不会分配太大,所以Eden区的GC会频繁的进行.因此在这里一般使用速度快,效率高的算法,使Eden尽快腾出空间.
2):**Full GC:**当年老代被写满,或者持久代被写满,或者显示调用System.gc(),再或者上一次GC之后Heap的各域分配策略动态变化时触发.
   Full GC 是对整个堆进行整理,包括Young,Tenured和Perm
   因为是对整个堆进行回收,因此要比Minor GC要慢很多,所以要尽可能的减少Full GC的次数
   在JVM调优的过程中,很大一部分工作都是对Full GC调节.

### 五.垃圾回收算法 ###
