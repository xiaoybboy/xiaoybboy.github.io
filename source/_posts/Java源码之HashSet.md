---
title: Java源码之HashSet
date: 2017-04-07 11:10:20
tags: [java,HashSet]
categories: java源码
---
# 概述
1.HashSet 是一个没有重复元素的集合。它是由HashMap实现的，不保证元素的顺序，特别是它不保证该顺序恒久不变。而且HashSet允许使用 null 元素。
2.HashSet是非同步的。如果多个线程同时访问一个哈希 set，而其中至少一个线程修改了该 set，那么它必须保持外部同步。这通常是通过对自然封装该 set 的对象执行同步操作来完成的。如果不存在这样的对象，则应该使用 Collections.synchronizedSet 方法来“包装” Set。最好在创建时完成这一操作，以防止对该 set 进行意外的不同步访问：
<!--more-->
```java
Set s = Collections.synchronizedSet(new HashSet(...));
```
3.HashSet通过iterator()返回的迭代器是fail-fast的。
4.HashSet的继承关系如下：
```java
	public class HashSet<E>  
	    extends AbstractSet<E>  
	    implements Set<E>, Cloneable, java.io.Serializable
```
# HashSet实现
HashSet是基于HashMap实现的，底层使用HashMap来保存所有元素，相关HashSet的操作，基本上都是直接调用底层HashMap的相关方法来完成.
## HashSet属性
```java
1.	private transient HashMap<E,Object> map;  
2.	  // Dummy value to associate with an Object in the backing Map 
3.	// 定义一个虚拟的Object对象作为HashMap的value，将此对象定义为static final。  
4.	private static final Object PRESENT = new Object(); 
```
可以看到，HashSet 实际上是使用 HashMap 来保存数据的。而且主要用的是 HashMap的 key。
PRESENT是什么东西呢？看上面的注释。dummy的意思是 挂名代表，傀儡。所以，可知：
PRESENT是向map中插入key-value对应的value
因为HashSet中只需要用到key，而HashMap是key-value键值对；
所以，向map中添加键值对时，键值对的值固定是PRESENT。每个set集合
中的元素都是HashMap的key 值（这也就保证了HashSet集合中不能有重复元素），而	它们的value值都是 PRESENT。
## HashSet构造函数
```java
1.	 /**  
2.	     * 默认的无参构造器，构造一个空的HashSet。
3.	     *  
4.	     * 实际底层会初始化一个空的HashMap，并使用默认初始容量为16和加载因子0.75。  
5.	     */    
6.	    public HashSet() {    
7.	    		map = new HashMap<E,Object>();    
8.	    }    
9.	    /**  
10.	     * 构造一个包含指定collection中的元素的新set。  
11.	     * 实际底层使用默认的加载因子0.75和足以包含指定  
12.	     * collection中所有元素的初始容量来创建一个HashMap。  
13.	     * @param c 其中的元素将存放在此set中的collection。  
14.	     */    
15.	    public HashSet(Collection<? extends E> c) {    
16.	     //为什么是Math.max((int) (c.size()/.75f) + 1, 16)？
17.	    //实际上默认的HashMap的加载因子是0.75,c.size()/0.75 就是HashMap的实际容量，而 16 是默认的HashMap的初始容量。所以取两者的较大值作为 HashSet的容量。
18.	    map = new HashMap<E,Object>(Math.max((int) (c.size()/.75f) + 1, 16));   
19.	    // 使用Collection实现的Iterator迭代器，将集合c的元素一个个加入HashSet中
20.	    	addAll(c);    
21.	   }    
22.	    /**  
23.	     * 以指定的initialCapacity和loadFactor构造一个空的HashSet。  
24.	     *  
25.	     * 实际底层以以指定的initialCapacity和loadFactor构造一个空的HashMap。  
26.	     * @param initialCapacity 初始容量。  
27.	     * @param loadFactor 加载因子。  
28.	    */    
29.	    public HashSet(int initialCapacity, float loadFactor) {    
30.	   		map = new HashMap<E,Object>(initialCapacity, loadFactor);    
31.	    }    
32.	    /**  
33.	     * 以指定的initialCapacity构造一个空的HashSet。  
34.	    *  
35.	     * 实际底层以相应的参数及加载因子loadFactor为0.75构造一个空的HashMap。  
36.	     * @param initialCapacity 初始容量。  
37.	     */    
38.	   public HashSet(int initialCapacity) {    
39.	    		map = new HashMap<E,Object>(initialCapacity);    
40.	    }    
41.	    /**  
42.	     * 以指定的initialCapacity和loadFactor构造一个新的空链接哈希集合。  
43.	    * 此构造函数为包访问权限，不对外公开，实际只是是对LinkedHashSet的支持。  
44.	     *  
45.	     * 实际底层会以指定的参数构造一个空LinkedHashMap实例来实现。  
46.     * @param initialCapacity 初始容量。  
47.	     * @param loadFactor 加载因子。  
48.     * @param dummy 标记。  没有实际意义
49.	    */    
50.	   HashSet(int initialCapacity, float loadFactor, boolean dummy) {    
51.   map = new LinkedHashMap<E,Object>(initialCapacity, loadFactor);    
52.	    } 
```
## HashSet方法
因为HashSet底层是HashMap实现，所以它的方法大都是直接调用HashMap的方法。
### add(),remove()
```java
1.	// 将元素(e)添加到HashSet中，也就是将元素作为Key放入HashMap中public boolean add(E e) {  
2.	      return map.put(e, PRESENT)==null;  //可以看到PRESENT是value值。
3.	  }  
4.	//  删除HashSet中的元素(o)，其实是在HashMap中删除了以o为key的Entry
5.	public boolean remove(Object o) {  
6.	      return map.remove(o)==PRESENT;  
7.	  }  
1.	  // 清空HashMap,的clear方法清空所有Entry
2.	public void clear() {
3.	        map.clear();    
4.	  }
```
具体可参考HashMap的源码。
### iterator(),size()，isEmpty()
```java
1.	public Iterator<E> iterator() {  
2.	     // 实际上返回的是HashMap的“key集合”的迭代器
3.	       return map.keySet().iterator();  
4.	   }  
5.	public int size() {  
6.	       return map.size();  
7.	   }  
8.	 public boolean isEmpty() {  
9.	       return map.isEmpty(); //查看map是否是空 
10.	   }  
11.	   public boolean contains(Object o) {  
12.	       return map.containsKey(o); //map中是否包含键 0 
13.	   }
```
看名字很容易就知道是干嘛的，都是直接用的HashMap的方法。
### clone()
```java
1.	/** 
2.	     * 返回此HashSet实例的浅表副本：并没有复制这些元素本身。 
3.	    * 底层实际调用HashMap的clone()方法，获取HashMap的浅表副本，并设置到  HashSet中。 
4.	     */  
5.	public Object clone() {  
6.	      try {  
7.	          HashSet<E> newSet = (HashSet<E>) super.clone();  
8.	          newSet.map = (HashMap<E, Object>) map.clone();  
9.	          return newSet;  
10.	      } catch (CloneNotSupportedException e) {  
11.	          throw new InternalError(e);  
12.	      }  
13.	  }
```
### 序列化
```java
1.	     // java.io.Serializable的写入流中 
2.	    // 将HashSet的“总的容量，加载因子，实际容量，所有的元素”都写入到输出流中  
3.	    private void writeObject(java.io.ObjectOutputStream s)  
4.	        throws java.io.IOException {  
5.	        // Write out any hidden serialization magic  
6.	        s.defaultWriteObject();  
7.	        s.writeInt(map.capacity());  
8.	        s.writeFloat(map.loadFactor());  
9.	        // Write out size  
10.	        s.writeInt(map.size());  
11.	        // Write out all elements in the proper order.  
12.	        for (Iterator i=map.keySet().iterator(); i.hasNext(); )  
13.	            s.writeObject(i.next());  
14.	    }  
15.	    // java.io.Serializable 从流中读取 HashSet对象 
16.	    // 将HashSet的“总的容量，加载因子，实际容量，所有的元素”依次读出  
17.	    private void readObject(java.io.ObjectInputStream s)  
18.	        throws java.io.IOException, ClassNotFoundException {  
19.	        // Read in any hidden serialization magic  
20.	        s.defaultReadObject();  
21.	 
22.	        int capacity = s.readInt();  
23.	        float loadFactor = s.readFloat();  
24.	        map = (((HashSet)this) instanceof LinkedHashSet ?  
25.	               new LinkedHashMap<E,Object>(capacity, loadFactor) :  
26.	               new HashMap<E,Object>(capacity, loadFactor));  
27.	        // Read in size  
28.	        int size = s.readInt();  
29.	        // Read in all elements in the proper order.  
30.	        for (int i=0; i<size; i++) {  
31.	            E e = (E) s.readObject();  
32.	            map.put(e, PRESENT);  
33.	        }  
34.	    }
```
# 总结
  总体来说，HashSet是比较简单的，底层是HashMap实现的，使用的都是HashMap的方法。
## HashSet遍历：
```java
      //遍历  
2.	        Iterator iterator = set.iterator();  
3.	        while (iterator.hasNext()) {  
4.	            System.out.println(iterator.next());              
5.	        }     
6.	          
7.	        //或者这样  
8.	        for (String s:set) {  
9.	            System.out.println(s);  
10.	        }
```
要注意的是：当我们要将一个类作为HashMap的key或者存储在HashSet的时候。通过重写hashCode()和equals(Object object)方法很重要，并且保证这两个方法的返回值一致。当两个类的hashCode()返回一致时，应该保证equasl()方法也返回true。
HashMap源码整理中...
