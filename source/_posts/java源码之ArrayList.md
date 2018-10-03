---
title: java源码之ArrayList
date: 2017-04-06 15:20:38
tags: [java源码,ArrayList]
categories: java集合
---
# ArrayList概述：  
```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable  
```
ArrayList是基于数组实现的，Object[] elementData;是一个动态数组，其容量能自动增长。
<!--more-->
ArrayList与Collection关系如下图：
　　 ![ArrayList和Collection](http://i.imgur.com/10g3wby.jpg)

１．ArrayList不是线程安全的，只能用在单线程环境下，多线程环境下可以考虑用Collections.synchronizedList(List l)函数返回一个线程安全的ArrayList类，也可以使用concurrent并发包下的CopyOnWriteArrayList类。
２．ArrayList实现了Serializable接口，因此它支持序列化，能够通过序列化传输，实现了RandomAccess接口，支持快速随机访问，实际上就是通过下标序号进行快速访问，实现了Cloneable接口，能被克隆。
３．每个ArrayList实例都有一个容量，该容量是指用来存储列表元素的数组的大小。它总是至少等于列表的大小。随着向ArrayList中不断添加元素，其容量也自动增长。自动增长会带来数据向新数组的重新拷贝，因此，如果可预知数据量的多少，可在构造ArrayList时指定其容量。在添加大量元素前，应用程序也可以使用ensureCapacity操作来增加ArrayList实例的容量，这可以减少递增式再分配的数量。 
注意，此实现不是同步的。如果多个线程同时访问一个ArrayList实例，而其中至少一个线程从结构上修改了列表，那么它必须保持外部同步。 
# ArrayList的实现：
对于ArrayList而言，它实现List接口、底层使用数组保存所有元素。其操作基本上是对数组的操作。下面我们来分析ArrayList的源代码：
## 私有属性：
ArrayList定义只定义类两个私有属性：
```java
/**  
* The array buffer into which the elements of the ArrayList are stored
* The capacity of the ArrayList is the length of this array buffer.  
 */    
private transient Object[] elementData;    

/**  
* The size of the ArrayList (the number of elements it contains).  
* @serial  
*/    
private int size;  
```
很容易理解，elementData存储ArrayList内的元素，size表示它包含的元素的数量。
有个关键字需要解释：transient。  
Java的serialization提供了一种持久化对象实例的机制。当持久化对象时，可能有一个特殊的对象数据成员，我们不想用serialization机制来保存它。为了在一个特定对象的一个域上关闭serialization，可以在这个域前加上关键字transient。被标记为transient的属性在对象被序列化的时候不会被保存。
接着回到ArrayList的分析中......
## modCount和Array.copyof()，System.arraycopy。
在父类AbstractList中定义了一个int型的属性：modCount，记录了ArrayList结构性变化的次数。
```java
protected transient int modCount = 0;
```
在ArrayList的所有涉及结构变化的方法中都增加modCount的值，包括：add()、remove()、addAll()、removeRange()及clear()方法。这些方法每调用一次，modCount的值就加1。
注：add()及addAll()方法的modCount的值是在其中调用的ensureCapacity()方法中增加的。 
然后，先了解一个方法，下面到处都会用到。Array.copyof()，System.arraycopy。
```java
	public static <T> T[] copyOf(T[] original, int newLength) {    
	    return (T[]) copyOf(original, newLength, original.getClass());    
	}    
	    
	public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {    
	    T[] copy = ((Object)newType == (Object)Object[].class)    
	        ? (T[]) new Object[newLength]   // 类型相同，则重新生成一个大小为newLength的数组实例    
	        : (T[]) Array.newInstance(newType.getComponentType(), newLength);  // 类型不同，重新生成一个大小为newLength的新类型数组实例    
	    System.arraycopy(original, 0, copy, 0, Math.min(original.length, newLength)); //将原数组内容拷贝到新数组中,新数组取最小的数组长度    
	    return copy;  // 返回新数组的引用     
	} 
```
而其中的System.arraycopy(Object src, int srcPos, Object dest, int destPos, int length)的声明：
```java  
public static native void arraycopy(Object src,  int  srcPos,  
                                        Object dest, int destPos,  
                                        int length);  
```
src - 源数组。 
srcPos - 源数组中的起始位置。 
dest - 目标数组。 
destPos - 目标数据中的起始位置。 
length - 要复制的数组元素的数量。
System.arraycopy是一个native方法，是底层使用c++实现的。 
## 构造方法：  
ArrayList提供了三种方式的构造器，可以构造一个默认的空列表、构造一个指定初始容量的空列表以及构造一个包含指定collection的元素的列表，这些元素按照该collection的迭代器返回它们的顺序排列的。
```java
	/** 
	     * Constructs an empty list with the specified initial capacity. 
	     */  
	    public ArrayList(int initialCapacity) {  
                 if (initialCapacity > 0) {  //初始容量大于0,实例化数组
	            this.elementData = new Object[initialCapacity];  
	        } else if (initialCapacity == 0) {  //初始容量为0
	            this.elementData = EMPTY_ELEMENTDATA;  //空数组
	        } else {  
	            throw new IllegalArgumentException("Illegal Capacity: "+  
	                                               initialCapacity);  
	        }  
	    }  
	  
	    /** ArrayList无参构造函数。
	     * Constructs an empty list with an initial capacity of ten. 
	     */  
	    public ArrayList() {  
	        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;  
	    }  
	  
	    /** 创建一个包含collection的ArrayList
	     * Constructs a list containing the elements of the specified 
	     * collection, in the order they are returned by the collection's 
	     * iterator.      */  
	    public ArrayList(Collection<? extends E> c) {  
          elementData = c.toArray();  //先转化成数组
	        if ((size = elementData.length) != 0) {  //如果集合不为空
	          // c.toArray might (incorrectly) not return Object[] 
	          if (elementData.getClass() != Object[].class)  
	  			//给elementData 赋值
	             elementData = Arrays.copyOf(elementData, size, Object[].class);
	        } else {  
	            // 如果是一个空的集合，用空数组来替换
	            this.elementData = EMPTY_ELEMENTDATA;  
	        }  
	    }
```
有参的两个构造器很好理解，下面说下无参的构造器。
EMPTY_ELEMENTDATA是什么的？看名字就知道了，是一个空的数组。DEFAULTCAPACITY_EMPTY_ELEMENTDATA也是一个空数组。那么他们之间有什么区别呢？为什么ArrayList无参构造函数构造的是一个DEFAULTCAPACITY_EMPTY_ELEMENTDATA的空数组呢？因为以前的代码是直接初始化一个长度为10的数组。看上面的注释，我们可以知道，We distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when first element is added.就是当第一个元素被加入到elementData中时，区分这两者。
```java
	/**
	    * 默认的初始容量为10 
	    */  
	   private static final int DEFAULT_CAPACITY = 10;  
	  
	   /** 
	    * Shared empty array instance used for empty instances. 
	    */  
	   private static final Object[] EMPTY_ELEMENTDATA = {};  
	  
	   /** 
	    * Shared empty array instance used for default sized empty instances. We
	    * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when  first element is added. 
	    */  
	   private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
```

既然如此，我们先看一下add()

```java
1.	public boolean add(E e) {  
2.	       ensureCapacityInternal(size + 1);  // Increments modCount!!  
3.	       elementData[size++] = e;  
4.	       return true;  
5.	   } 
```
看第一行，应该就是区分的体现了，下面着重看下这个ensureCapacityInternal()函数。这涉及到下面的数组容量扩充，我们单独说一下。
## 调整数组容量ensureCapacity： 
与之前的ArrayList源码改变最大的就是这一部分了，先把三个调整容量的函数都贴出来，下面要多次用到。ensureCapacityInternal是第二个。
```java
1.	/** 
2.	     * Increases the capacity of this <tt>ArrayList</tt> instance, if 
3.	     * necessary, to ensure that it can hold at least the number of elements 
4.	     * specified by the minimum capacity argument. 
5.	     * 
6.	     * @param   minCapacity   the desired minimum capacity 
7.	     */  
8.	    public void ensureCapacity(int minCapacity) {  
9.	        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)  
10.	            // any size if not default element table  
11.	            ? 0  
12.	            // larger than default for default empty table. It's already  
13.	            // supposed to be at default size.  
14.	            : DEFAULT_CAPACITY;  
15.	  
16.	        if (minCapacity > minExpand) {  
17.	            ensureExplicitCapacity(minCapacity);  
18.	        }  
19.	    }  
20.	  
21.	    private void ensureCapacityInternal(int minCapacity) {  
22.	        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {  
23.	            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);  
24.	        }  
25.	        ensureExplicitCapacity(minCapacity);  
26.	    }  
27.	  
28.	    private void ensureExplicitCapacity(int minCapacity) {  
29.	        modCount++;   
30.	        if (minCapacity - elementData.length > 0)  
31.	            grow(minCapacity);  
32.	    }  

1.	   /**  容量扩充，确保数组中至少能包含minimum capacity个元素
2.	     * Increases the capacity to ensure that it can hold at least the 
3.	     * number of elements specified by the minimum capacity argument. 
4.	     * 
5.	     * @param minCapacity the desired minimum capacity 
6.	     */  
7.	    private void grow(int minCapacity) {  
8.	        int oldCapacity = elementData.length;  
9.	      int newCapacity = oldCapacity + (oldCapacity >> 1); //原来容量的1.5倍 
10.	        if (newCapacity - minCapacity < 0)  
11.	            newCapacity = minCapacity;  
12.	        if (newCapacity - MAX_ARRAY_SIZE > 0)  
13.	            newCapacity = hugeCapacity(minCapacity);  
14.	        // minCapacity is usually close to size, so this is a win:  
15.	        elementData = Arrays.copyOf(elementData, newCapacity);  
16.	    }  
17.	  //对大容量数组的处理
18.	    private static int hugeCapacity(int minCapacity) {  
19.	        if (minCapacity < 0) // overflow  
20.	            throw new OutOfMemoryError();  
21.	        return (minCapacity > MAX_ARRAY_SIZE) ?  
22.	            Integer.MAX_VALUE :  
23.	            MAX_ARRAY_SIZE;  
24.	    }
```
看ensureCapacityInternal中的下面这一句
```java
33.	if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {  
34.	            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);  
35.	        } 
```
看到了吧，先比较DEFAULT_CAPACITY和minCapacity，取较大的值作为minCapacity，显然，对于无参的构造器的空数组，比较的结果是DEFAULT_CAPACITY(上面有定义是10)，然后接下来执行ensureExplicitCapacity(minCapacity)。再接下来，实际执行扩充的是grow(int minCapacity) 这个函数。上面如果是初始化的时候，容易计算此时的minCapacity = DEFAULT_CAPACITY = 10.所以执行grow的时候，
```java
1.	if (newCapacity - minCapacity < 0)  
            newCapacity = minCapacity;
```
会执行这一句，应该空的数组扩充1.5倍之后还是空的。接着往下看，就可以知道其实无参的ArrayList的构造器初始化的是一个长度为10的elementData 数组。
### 总结
对于无参的ArrayList构造器，初始化的其实是一个容量为10的Object[] elementData 数组。每次执行扩充时，最终进行数组容量扩充的是grow（）函数，而且不难看出每次数组扩充后的容量为原来数组容量 的1.5倍。int newCapacity = oldCapacity + (oldCapacity >> 1);
## 元素存储：
ArrayList提供了set(int index, E element)、add(E e)、add(int index, E element)、addAll(Collection<? extends E> c)、addAll(int index, Collection<? extends E> c)这些添加元素的方法。下面我们一一讲解：
```java
1.	  //两个范围检验函数，查看索引是否越界，越界抛出异常  
2.	  private void rangeCheck(int index) {    
3.	          if (index >= size)    
4.	              throw new IndexOutOfBoundsException(outOfBoundsMsg(index));  
5.	      }    
6.	      private void rangeCheckForAdd(int index) {    
7.	          if (index > size || index < 0)    
8.	              throw new IndexOutOfBoundsException(outOfBoundsMsg(index));   
9.	.     }    
10.	
11.	   // 用指定的元素替代此列表中指定位置上的元素，并返回以前位于该位置上的元素。      
12.	   public E set(int index, E element) {      
13.	      RangeCheck(index);      
14.	      E oldValue = (E) elementData[index];      
15.	      elementData[index] = element;      
16.	      return oldValue;      
17.	   }        
18.	   // 将指定的元素添加到此列表的尾部。 上面讲到过这个函数，实现也很简单     
19.	   public boolean add(E e) {  
20.	         ensureCapacityInternal(size + 1);  // Increments modCount!!  
21.	         elementData[size++] = e;  
22.	         return true;  
23.	      }      
24.	   // 将指定的元素插入此列表中的指定位置。      
25.	   // 如果当前位置有元素，则向右移动当前位于该位置的元素以及所有后续元素（将其索引加1）。  
26.	   public void add(int index, E element) {  
27.	          rangeCheckForAdd(index);//索引范围检验  
28.	          ensureCapacityInternal(size + 1);  //  扩充1个单元  
29.	       // 将 elementData中从Index位置开始、长度为size-index的元素，      
30.	      // 拷贝到从下标为index+1位置开始的新的elementData数组中。      
31.	      // 即将当前位于该位置的元素以及所有后续元素右移一个位置。  
32.	         System.arraycopy(elementData, index, elementData, index + 1,  
33.	                          size - index);  
34.	         elementData[index] = element;//插入元素  
35.	         size++;  
36.	     }      
37.	       
38.	  // 按照指定collection的迭代器所返回的元素顺序，将该collection中的所有元素添加到此列表的尾部。  
39.	 public boolean addAll(Collection<? extends E> c) {  
40.	         Object[] a = c.toArray();//先转换成数组   
41.	         int numNew = a.length;  
42.	        ensureCapacityInternal(size + numNew);  // 容量扩充            
43.	         System.arraycopy(a, 0, elementData, size, numNew);//元素复制  
44.	        size += numNew;  
45.	         return numNew != 0;  
46.	      }           
47.	   // 从指定的位置开始，将指定collection中的所有元素插入到此列表中。  
48.	 public boolean addAll(int index, Collection<? extends E> c) {  
49.	          rangeCheckForAdd(index);//索引检查  
50.	    
51.	          Object[] a = c.toArray();  
52.	          int numNew = a.length;  
53.	          ensureCapacityInternal(size + numNew);  // 容量扩充  
54.	    
55.	         int numMoved = size - index;//要移动的元素的个数  
56.	         if (numMoved > 0)  
             System.arraycopy(elementData, index, elementData, index + numNew, numMoved);//复制元素  
58.	   
59.	         System.arraycopy(a, 0, elementData, index, numNew);  
60.	         size += numNew;  
61.	         return numNew != 0;  
62.	    }  
```
ArrayList是基于数组实现的，可以看到上面的添加元素的方法，大都需要先进行索引检查（与操作索引相关的），还要进行数组容量的扩充，然后是 System.arraycopy复制元素。具体过程查看上面的注释。
## 元素读取：
这个很简单，直接返回数组i位置上的元素即可。
```java
	// 返回此列表中指定位置上的元素。    
	 public E get(int index) {    
	    RangeCheck(index);    
	    return (E) elementData[index];    
	  }   
 ```
## 元素删除：
ArrayList提供了根据下标或者指定对象两种方式的删除功能。如下：
### romove(int index):
```java
	 // 移除此列表中指定位置上的元素。    
	   public E remove(int index) {    
	      RangeCheck(index);    
	      modCount++;    
	      E oldValue = (E) elementData[index];    
	      int numMoved = size - index - 1;    
	      if (numMoved > 0)    
	         System.arraycopy(elementData, index+1, elementData, index, numMoved);    
	     elementData[--size] = null; // Let gc do its work    
	     
	     return oldValue;    
	  } 
```
首先是检查范围，修改modCount，保留将要被移除的元素，将移除位置之后的元素向前挪动一个位置，将list末尾元素置空（null），返回被移除的元素。
### remove(Object o)
```java
  // 移除此列表中首次出现的指定元素（如果存在）。这是应为ArrayList中允许存放重复的元素。    
  public boolean remove(Object o) {    
     // 由于ArrayList中允许存放null，因此下面通过两种情况来分别处理。    
     if (o == null) {    
         for (int index = 0; index < size; index++)    
            if (elementData[index] == null) {    
                 // 类似remove(int index)，移除列表中指定位置上的元素。    
                 fastRemove(index);    
                 return true;    
             }    
     } else {    
         for (int index = 0; index < size; index++)    
            if (o.equals(elementData[index])) {    
                 fastRemove(index);    
                 return true;    
             }    
         }    
         return false;    
     }   
 }
```
首先通过代码可以看到，当移除成功后返回true，否则返回false。remove(Object o)中通过遍历element寻找是否存在传入对象，一旦找到就调用fastRemove移除对象。为什么找到了元素就知道了index，不通过remove(index)来移除元素呢？因为fastRemove跳过了判断边界的处理，因为找到元素就相当于确定了index不会超过边界，而且fastRemove并不返回被移除的元素。下面是fastRemove的代码，基本和remove(index)一致。方法说明：skips bounds checking and does notreturn the value removed.
```java
 private void fastRemove(int index) {    
          modCount++;    
          int numMoved = size - index - 1;    
          if (numMoved > 0)    
   System.arraycopy(elementData, index+1, elementData, index,    numMoved);
          elementData[--size] = null; // Let gc do its work    
  }  
```
### batchRemove()
```java
1.	//保留集合c中的元素，集合外的元素全部删除
2.	      Retains only the elements in this list that are contained in the
3.	     * specified collection.  In other words, removes from this list all
4.	     * of its elements that are not contained in the specified collection.
5.	public boolean retainAll(Collection<?> c) {  
6.	        Objects.requireNonNull(c);  
7.	        return batchRemove(c, true);  
8.	    }  
9.	  
10.	    private boolean batchRemove(Collection<?> c, boolean complement) {  
11.	        final Object[] elementData = this.elementData;  
12.	        int r = 0, w = 0;  
13.	        boolean modified = false;  
14.	        try {  
15.	            for (; r < size; r++)  
16.	                if (c.contains(elementData[r]) == complement)  
17.	                    elementData[w++] = elementData[r];  
18.	        } finally {  
19.	            // Preserve behavioral compatibility with AbstractCollection,  
20.	            // even if c.contains() throws.  
21.	            if (r != size) {  
22.	                System.arraycopy(elementData, r,  
23.	                                 elementData, w,  
24.	                                 size - r);  
25.	                w += size - r;  
26.	            }  
27.	            if (w != size) {  
28.	                // clear to let GC do its work  
29.	                for (int i = w; i < size; i++)  
30.	                    elementData[i] = null;  
31.	                modCount += size - w;  
32.	                size = w;  
33.	                modified = true;  
34.	            }  
35.	        }  
36.	        return modified;  
37.	    }  
```
### removeRange(int fromIndex,int toIndex)
```java
	 protected void removeRange(int fromIndex, int toIndex) {    
      modCount++;    
      int numMoved = size - toIndex;    
          System.arraycopy(elementData, toIndex, elementData, fromIndex,    
                           numMoved);    
      
      // Let gc do its work    
      int newSize = size - (toIndex-fromIndex);    
     while (size != newSize)    
          elementData[--size] = null;    
 }  
```
执行过程是将elementData从toIndex位置开始的元素向前移动到fromIndex，然后将toIndex位置之后的元素全部置空顺便修改size。
这个方法是protected，及受保护的方法，为什么这个方法被定义为protected呢？
这是一个解释，但是可能不容易看明白。http://stackoverflow.com/questions/2289183/why-is-javas-abstractlists-removerange-method-protected
先看下面这个例子
1.	  ArrayList<Integer> ints = new ArrayList<Integer>(Arrays.asList(0, 1, 2,    
2.	         3, 4, 5, 6));    
3.	 // fromIndex low endpoint (inclusive) of the subList    
4.	 // toIndex high endpoint (exclusive) of the subList    
5.	ints.subList(2, 4).clear();    
6.	 System.out.println(ints);    
输出结果是[0, 1, 4, 5, 6]，结果是不是像调用了removeRange(int fromIndex,int toIndex)！哈哈哈，就是这样的。但是为什么效果相同呢？是不是调用了removeRange(int fromIndex,int toIndex)呢？
## ArrayList和Vector区别：
•	ArrayList在内存不够时默认是扩展50% + 1个，Vector是默认扩展1倍。
•	Vector提供indexOf(obj, start)接口，ArrayList没有。
•	Vector属于线程安全级别的，但是大多数情况下不使用Vector，因为线程安全需要更大的系统开销。
##　trimToSize
 ArrayList还给我们提供了将底层数组的容量调整为当前列表保存的实际元素的大小的功能。它可以通过trimToSiz方法来实现。代码如下：
```java
public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    } 
```
由于elementData的长度会被拓展，size标记的是其中包含的元素的个数。所以会出现size很小但elementData.length很大的情况，将出现空间的浪费。trimToSize将返回一个新的数组给elementData，元素内容保持不变，length和size相同，节省空间。
## 转为静态数组toArray
注意ArrayList的两个转化为静态数组的toArray方法。
第一个， 调用Arrays.copyOf将返回一个数组，数组内容是size个elementData的元素，即拷贝elementData从0至size-1位置的元素到新数组并返回.
```java
1.	public Object[] toArray() {    
2.	         return Arrays.copyOf(elementData, size);    
3.	 } 
```
第二个，如果传入数组的长度小于size，返回一个新的数组，大小为size，类型与传入数组相同。所传入数组长度与size相等，则将elementData复制到传入数组中并返回传入的数组。若传入数组长度大于size，除了复制elementData外，还将把返回数组的第size个元素置为空。
```java
	public <T> T[] toArray(T[] a) {  
	        if (a.length < size)  
	            // Make a new array of a's runtime type, but my contents:  
	            return (T[]) Arrays.copyOf(elementData, size, a.getClass());  
	    System.arraycopy(elementData, 0, a, 0, size);  
	        if (a.length > size)  
	            a[size] = null;  
	        return a;  
	    }
```
## clone(),
返回此 ArrayList 实例的浅表副本。（不复制这些元素本身。）调用父类的clone方法返回一个对象的副本，将返回对象的elementData数组的内容赋值为原对象elementData数组的内容，将副本的modCount设置为0。
```java
	public Object clone() {  
	        try {  
	            ArrayList<?> v = (ArrayList<?>) super.clone();  
	            v.elementData = Arrays.copyOf(elementData, size);  
	            v.modCount = 0;  
	            return v;  
	        } catch (CloneNotSupportedException e) {  
	            // this shouldn't happen, since we are Cloneable  
            throw new InternalError(e);  
	        }  
	    }
```
## clear()
```java
	public void clear() {  
	     modCount++;  
	   
	     // Let gc do its work  
	     for (int i = 0; i < size; i++)  
         elementData[i] = null;  
	   
	     size = 0;  
	     } 
```
clear的时候并没有修改elementData的长度（好不容易申请、拓展来的，凭什么释放，留着搞不好还有用呢。这使得确定不再修改list内容之后最好调用trimToSize来释放掉一些空间），只是将所有元素置为null，size设置为0。
## contains(Object)
```java
public boolean contains(Object o) {  
	return indexOf(o) >= 0;  
} 
```
1.indexOf方法返回值与0比较来判断对象是否在list中。接着看indexOf。
```java
	public int indexOf(Object o) {  
	        if (o == null) {  
	            for (int i = 0; i < size; i++)  
                if (elementData[i]==null)  
	                    return i;  
	        } else {  
	            for (int i = 0; i < size; i++)  
	                if (o.equals(elementData[i]))  
	                    return i;  
	        }  
	        return -1;  
    } 
```
2.lastIndexOf，光看名字应该就明白了返回的是传入对象在elementData数组中最后出现的index值
```java
	public int lastIndexOf(Object o) {  
	     if (o == null) {  
	         for (int i = size-1; i >= 0; i--)  
	         if (elementData[i]==null)  
	             return i;  
	     } else {  
	         for (int i = size-1; i >= 0; i--)  
	         if (o.equals(elementData[i]))  
	             return i;  
     }  
	     return -1;  
	     }
```
采用了从后向前遍历element数组，若遇到Object则返回index值，若没有遇到，返回-1.
## 序列化
因为实现了java.io.Serializable接口，索引可以进行序列化操作。
```java
	/** 
	     * Save the state of the <tt>ArrayList</tt> instance to a stream (that 
	     * is, serialize it). 把arraylist实例写入一个流中
	     */  
	    private void writeObject(java.io.ObjectOutputStream s)  
	        throws java.io.IOException{  
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
	  
	    /** 
	     * Reconstitute the <tt>ArrayList</tt> instance from a stream (that is, 
	     * deserialize it). 从流中读出一个ArrayList实例
	     */  
	    private void readObject(java.io.ObjectInputStream s)  
	        throws java.io.IOException, ClassNotFoundException {  
	        elementData = EMPTY_ELEMENTDATA;  
	  
	        // Read in size, and any hidden stuff  
	        s.defaultReadObject();  
 
	        // Read in capacity  
	        s.readInt(); // ignored  
	  
	        if (size > 0) {  
	            // be like clone(), allocate array based upon size not capacity  
	            ensureCapacityInternal(size);  
	  
	            Object[] a = elementData;  
	            // Read in all elements in the proper order.  
	            for (int i=0; i<size; i++) {  
	                a[i] = s.readObject();  
	        }  }
	    } 
```
# Fail-Fast机制： 
ArrayList也采用了快速失败的机制，通过记录modCount参数来实现。在面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。
# ArrayList的遍历方式
ArrayList支持3种遍历方式
```java
1、通过迭代器遍历：
Iterator iter = list.iterator();
while (iter.hasNext())
{
    System.out.println(iter.next());
}         
2、随机访问，通过索引值去遍历，由于ArrayList实现了RandomAccess接口
int size = list.size();
for (int i=0; i<size; i++) 
{
   System.out.println(list.get(i));        
}      
3、for循环遍历：
for(String str:list)
{
  System.out.println(str);
}
```
# 总结:
关于ArrayList的源码，给出几点比较重要的总结：
1.注意其三个不同的构造方法。无参构造方法构造的ArrayList的容量默认为10，带有Collection参数的构造方法，将Collection转化为数组赋给ArrayList的实现数组elementData。
2.注意扩充容量的方法ensureCapacity。ArrayList在每次增加元素（可能是1个，也可能是一组）时，都要调用该方法来确保足够的容量。当容量不足以容纳当前的元素个数时，就设置新的容量为旧的容量的1.5倍，如果设置后的新容量还不够，则直接新容量设置为传入的参数（也就是所需的容量），而后用Arrays.copyof()方法将元素拷贝到新的数组（详见下面的第3点）。从中可以看出，当容量不够时，每次增加元素，都要将原来的元素拷贝到一个新的数组中，非常之耗时，也因此建议在事先能确定元素数量的情况下，才使用ArrayList，否则建议使用LinkedList。
3.ArrayList的实现中大量地调用了Arrays.copyof()和System.arraycopy()方法。我们有必要对这两个方法的实现做下深入的了解。
首先来看Arrays.copyof()方法。它有很多个重载的方法，但实现思路都是一样的，我们来看泛型版本的源码：
```java
1.	public static <T> T[] copyOf(T[] original, int newLength) {    
2.	    return (T[]) copyOf(original, newLength, original.getClass());    
3.	}
```  
很明显调用了另一个copyof方法，该方法有三个参数，最后一个参数指明要转换的数据的类型，其源码如下：
```java
1.	public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {    
2.	    T[] copy = ((Object)newType == (Object)Object[].class)    
3.	        ? (T[]) new Object[newLength]    
4.	        : (T[]) Array.newInstance(newType.getComponentType(), newLength);    
5.	    System.arraycopy(original, 0, copy, 0,    
6.	                     Math.min(original.length, newLength));    
7.	    return copy;    
8.	} 
```  
这里可以很明显地看出，该方法实际上是在其内部又创建了一个长度为newlength的数组，调用System.arraycopy()方法，将原来数组中的元素复制到了新的数组中。
下面来看System.arraycopy()方法。该方法被标记了native，调用了系统的C/C++代码，在JDK中是看不到的，但在openJDK中可以看到其源码。该函数实际上最终调用了C语言的memmove()函数，因此它可以保证同一个数组内元素的正确复制和移动，比一般的复制方法的实现效率要高很多，很适合用来批量处理数组。Java强烈推荐在复制大量数组元素时用该方法，以取得更高的效率。
4.ArrayList基于数组实现，可以通过下标索引直接查找到指定位置的元素，因此查找效率高，但每次插入或删除元素，就要大量地移动元素，插入删除元素的效率低。
5.在查找给定元素索引值等的方法中，源码都将该元素的值分为null和不为null两种情况处理，ArrayList中允许元素为null。
6.数组扩容 
这是对ArrayList效率影响比较大的一个因素。 
每 当执行Add、AddRange、Insert、InsertRange等添加元素的方法，都会检查内部数组的容量是否不够了，如果是，它就会以当前容量 的两倍来重新构建一个数组，将旧元素Copy到新数组中，然后丢弃旧数组，在这个临界点的扩容操作，应该来说是比较影响效率的。
由于是从word上粘贴过来的，格式有些不太正确，以后慢慢修改吧。 
