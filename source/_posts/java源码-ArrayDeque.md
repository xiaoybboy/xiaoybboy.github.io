---
title: java源码--ArrayDeque
date: 2017-08-14 13:52:23
tags: [java,ArrayDeque]
categories: java源码
---
# 概述
Deque意为双端队列，ArrayDeque显然是基于数组实现的双端队列，而且作为双端队列时，效率比LinkList高。而且其特性使它还可以当做栈来使用，效率比Stack高。
ArrayDeque是非线程安全的（not thread-safe），当多个线程同时使用的时候，需要程序员手动同步；另外，该容器不允许放入null元素。
![](http://i.imgur.com/SRn9K5h.png)
<!--more-->
继承体系：
```java
	public class ArrayDeque<E> extends AbstractCollection<E>  
	                           implements Deque<E>, Cloneable, Serializable  
```
# 存储结构
从内部存储上，可以看到内部只有一个数组
```java
	    // 底层用数组存储元素  
	     private transient E[] elements;  
	     // 队列的头部元素索引（即将pop出的一个）  
	      private transient int head;  
	     // 队列下一个要添加的元素索引 ，注意是下一个，不是最后一个元素
	      private transient int tail;  
	     // 最小的初始化容量大小，需要为2的n次幂  
	      private static final int MIN_INITIAL_CAPACITY = 8;  
```
![](http://i.imgur.com/sxw4t8w.png)
上图中我们看到，head指向首端第一个有效元素，tail指向尾端第一个可以插入元素的空位，不是当前数组的元素位置。
因为是循环数组，所以head不一定总等于0，tail也不一定总是比head大。
# 构造方法

```java
	     /** 
	     * 默认构造方法，数组的初始容量为16 
	     */  
	    public ArrayDeque() {  
	        elements = (E[]) new Object[16];  
	    }  
	    
	    /** 
	     * 使用一个指定的初始容量构造一个ArrayDeque,但是最终分配的容量并不是numElements，初始容量是大于指定numElements的最小的2的n次幂 
	
	     */  
	    public ArrayDeque( int numElements) {  
	        allocateElements(numElements);  
	    }  
	   
	    /** 
	     * 构造一个指定Collection集合参数的ArrayDeque 
	     */  
	    public ArrayDeque(Collection<? extends E> c) {  
	        allocateElements(c.size());  
	        addAll(c);  
	    }
```
 
allocateElements()是什么东西呢？

```java
	 /** 
	     * 分配合适容量大小的数组，确保初始容量是大于指定numElements的最小的2的n次幂 
	     */  
	    private void allocateElements(int numElements) {  
	        int initialCapacity = MIN_INITIAL_CAPACITY;  //初始容量为8
	        // 找到大于指定容量的最小的2的n次幂  
	        // Find the best power of two to hold elements.  
	        // Tests "<=" because arrays aren't kept full.  
	        // 如果指定的容量小于初始容量8，则执行一下if中的逻辑操作  
	        if (numElements >= initialCapacity) {  
	            initialCapacity = numElements;  
	            initialCapacity |= (initialCapacity >>>  1);  
	            initialCapacity |= (initialCapacity >>>  2);  
	            initialCapacity |= (initialCapacity >>>  4);  
	            initialCapacity |= (initialCapacity >>>  8);  
	            initialCapacity |= (initialCapacity >>> 16);  
	            initialCapacity++;  
	   
	            if (initialCapacity < 0)   // Too many elements, must back off  
	                initialCapacity >>>= 1; // Good luck allocating 2 ^ 30 elements  
	        }  
        elements = (E[]) new Object[initialCapacity];  
	    }
```
 
此方法是给数组分配初始容量，初始容量并不是numElements，而是大于指定长度的最小的2的幂正数
所以ArrayDeque的容量一定是2的幂整数.至于这是为什么，应该是计算机内存分配的关系吧。2的幂次方页空间分配的更快。（个人猜测）
# 重要方法
## 入队（添加元素到队尾）

```java
	    /** 
	     * 增加一个元素，如果队列已满，则抛出一个IIIegaISlabEepeplian异常 
	     */  
	    public boolean add(E e) {  
	        // 调用addLast方法，将元素添加到队尾  
	        addLast(e);  
	        return true;  
	    }  
	* 把元素填入到队列前端.即在head的前面添加元素 
	*/  
	  public void addFirst(E e) {  
	    if (e == null)  //允许插入空值
	       throw new NullPointerException();  
	   elements[head = (head - 1) & (elements.length - 1)] = e;  
	   if (head == tail)  //表示队列已经满了
	       doubleCapacity();  //扩充容量
	}
	      /** 
	     * 将元素添加到队尾 
	     */  
	    public void addLast(E e) {  
        // 如果元素为null，咋抛出空指针异常  
        if (e == null)  
	            throw new NullPointerException();  
        // 将元素e放到数组的tail位置  
	        elements[tail] = e;  
	        // 判断tail和head是否相等，如果相等则对数组进行扩容  
	        if ( (tail = (tail + 1) & ( elements.length - 1)) == head)  
	            // 进行两倍扩容  
            doubleCapacity();  
	    }  
     /** 
	     * 添加一个元素 
	     */  
	    public boolean offer(E e) {  
	        // 调用offerLast方法，将元素添加到队尾  
	        return offerLast(e);  
	    }  
	     /** 
	     * Inserts the specified element at the front of this deque. 
	     */  
	    public boolean offerFirst(E e) {  
	        addFirst(e);  
	        return true;  
	    } 
	    /** 
	     * 在队尾添加一个元素 
	     */  
	    public boolean offerLast(E e) {  
	        // 调用addLast方法，将元素添加到队尾  
	        addLast(e);  
	        return true;  
	    }    
```
  
## addFirst()
addFirst(E e)的作用是在Deque的首端插入元素，也就是在head的前面插入元素，在空间足够且下标没有越界的情况下，只需要将elements[--head] = e即可。
![](http://i.imgur.com/6oMsBPQ.png)
实际需要考虑：1.空间是否够用 2.下标是否越界的问题。上图中，如果head为0之后接着调用addFirst()，虽然空余空间还够用，但head为-1，下标越界了。下列代码很好的解决了这两个问题.

```java
	* 把元素填入到队列前端.即在head的前面添加元素 
	*/  
	  public void addFirst(E e) {  
	    if (e == null)  //允许插入空值
	       throw new NullPointerException();  
	   elements[head = (head - 1) & (elements.length - 1)] = e;  
	   if (head == tail)  //表示队列已经满了
        doubleCapacity();  //扩充容量
	}
```

上述代码我们看到，空间问题是在插入之后解决的，因为tail总是指向下一个可插入的空位，也就意味着elements数组至少有一个空位，所以插入元素的时候不用考虑空间问题。
下标越界的处理解决起来非常简单，head = (head - 1) & (elements.length - 1)就可以了，这段代码相当于取余，同时解决了head为负值的情况。因为elements.length必需是2的指数倍，elements - 1就是二进制低位全1，跟head - 1相与之后就起到了取模的作用，如果head - 1为负数（其实只可能是-1），则相当于对其取相对于elements.length的补码。
比如说，如果elements.length为8，则(elements.length - 1)为7，二进制为0111，对于负数-1，与7相与，结果为7，对于正数8，与7相与，结果为0，都能达到循环数组中找下一个正确位置的目的。
对于取与操作而言，有下面的结论：
```
	记mod = elements.length = 2^k, a为[-1,module+1]之间的一个整数，那么有：  
	  
	a == -1:  
			a & (mod-1) == mod - 1;  
	0 <= a < mod:  
			a & (mod - 1) == a  
	a == mod:  
			a & (mod - 1) == 0 
```
下面再说说扩容函数doubleCapacity()，其逻辑是申请一个更大的数组（原数组的两倍），然后将原数组复制过去。过程如下图所示：
 ![](http://i.imgur.com/TXjfxNm.png)
图中我们看到，复制分两次进行，第一次复制head右边的元素，第二次复制head左边的元素。

```java
	    /** 将队列的容量变成二倍
	    * Doubles the capacity of this deque.  Call only when full, i.e., 
	    * when head and tail have wrapped around to become equal. 
	    */  
	   private void doubleCapacity() {  
	       assert head == tail;  
	       int p = head;  
	       int n = elements.length;  
	       int r = n - p; // number of elements to the right of p  
	       int newCapacity = n << 1;  
	       if (newCapacity < 0)  
	           throw new IllegalStateException("Sorry, deque too big");  
       Object[] a = new Object[newCapacity];  
	       System.arraycopy(elements, p, a, 0, r);  
	       System.arraycopy(elements, 0, a, r, p);  
	       elements = a;  
	       head = 0;  
       tail = n; 
```

## addLast()
addLast(E e)的作用是在Deque的尾端插入元素，也就是在tail的位置插入元素，由于tail总是指向下一个可以插入的空位，因此只需要elements[tail] = e;即可。插入完成后再检查空间，如果空间已经用光，则调用doubleCapacity()进行扩容。
 ![](http://i.imgur.com/OMfipDU.png)

```java
	   /** 
	    * Inserts the specified element at the end of this deque. 
	    */  
	   public void addLast(E e) {  
	       if (e == null)  
	           throw new NullPointerException();  
	       elements[tail] = e;  
	       if ( (tail = (tail + 1) & (elements.length - 1)) == head) //在tail位置后面加入元素 
	           doubleCapacity();  
   }
```
  
下标越界处理方式和addFirst()相同
## offerFirst(),offerLast()
这两个方法从源码看的很清楚，与addxx()方法的区别就是:add()方法插入失败(插入一个null)会抛出异常，而offer()方法至于插入成功时才会返回true，可用于某些判断。
## 出队（移除并返回队头元素）

```java
    /** 
	     * 移除并返回队列头部的元素，如果队列为空，则抛出一个NoSuchElementException异常 
	     */  
	    public E remove() {  
	        // 调用removeFirst方法，移除队头的元素  
        return removeFirst();  
	    }  
	   
	    /** 
	     * @throws NoSuchElementException {@inheritDoc} 
	     */  
	    public E removeFirst() {  
	        // 调用pollFirst方法，移除并返回队头的元素  
	        E x = pollFirst();  
	        // 如果队列为空，则抛出NoSuchElementException异常  
	        if (x == null)  
	            throw new NoSuchElementException();  
	        return x;  
	    }  
     /** 
	    * @throws NoSuchElementException {@inheritDoc} 
	    */  
   public E removeLast() {  
	       E x = pollLast();  
       if (x == null)  
	           throw new NoSuchElementException();  
	       return x;  
	   }     
	    /** 
	     * 移除并返问队列头部的元素，如果队列为空，则返回null 
	     */  
	    public E poll() {  
	        // 调用pollFirst方法，移除并返回队头的元素  
	        return pollFirst();  
	    }  
	
    public E pollFirst() {  
        int h = head ;  
	        // 取出数组队头位置的元素  
        E result = elements[h]; // Element is null if deque empty  
	        // 如果数组队头位置没有元素，则返回null值  
	        if (result == null)  
            return null;  
	        // 将数组队头位置置空，也就是删除元素  
        elements[h] = null;     // Must null out slot  
	        // 将head指针往前移动一个位置  
	        head = (h + 1) & (elements .length - 1);  
	        // 将队头元素返回  
	        return result;  
	    }  
	
	  public E pollLast() {  
       int t = (tail - 1) & (elements.length - 1);  //因为tail总是指向下一个队列元素，所以pollLast时候，实际取得是tail-1位置的元素
	       @SuppressWarnings("unchecked")  
	       E result = (E) elements[t];  
	       if (result == null)  //null值意味着deque为空
	           return null;  
	       elements[t] = null;  
	       tail = t;  
	       return result;  
	   }  
```

可以看到，remove方法和poll方法的区别是：如果没有出队元素(null)，remove方法抛出异常，而poll方法返回null;
## 返回队列元素(不出队)

```java
	    /** 
	     *  返回队列头部的元素，如果队列为空，则抛出一个NoSuchElementException异常 
	     */  
	    public E element() {  
	        // 调用getFirst方法，获取队头的元素  
	        return getFirst();  
	    }  
	   
	    /** 
	     * @throws NoSuchElementException {@inheritDoc} 
.	     */  
    public E getFirst() {  
	        // 取得数组head位置的元素  
	        E x = elements[head ];  
	        // 如果数组head位置的元素为null，则抛出异常  
	        if (x == null)  
            throw new NoSuchElementException();  
	        return x;  
	    }  
	     /** 
	     * @throws NoSuchElementException {@inheritDoc} 
	     */  
	    public E getLast() {  
        @SuppressWarnings("unchecked")  
        E result = (E) elements[(tail - 1) & (elements.length - 1)];  
	        if (result == null)  
	            throw new NoSuchElementException();  
	        return result;  
	    }  
	   
	    /** 
	     * 返回队列头部的元素，如果队列为空，则返回null 
	     .	     */  
    public E peek() {  
	        // 调用peekFirst方法，获取队头的元素  
	        return peekFirst();  
	    }  
	   
	    public E peekFirst() {  
       // 取得数组head位置的元素并返回  
	        return elements [head]; // elements[head] is null if deque empty  
	    }  
	  public E peekLast() {  
	      return (E) elements[(tail - 1) & (elements.length - 1)];  
  }  
```

get()和peek()方法的区别在于：get()方法取到的元素是null(队列为空)时，抛出异常，而peek()方法返回null;
## push(),pop()
显而易见的操作，把ArrayDeque当做堆栈使用时，push()即为在head处插入元素，pop（）即为删除head处元素。

```java
	public void push(E e) {  
	        addFirst(e);  
	    }  
	
	public E pop() {  
	        return removeFirst();  
	    }
```
 
## 其他

```java
	    /** 
	    * Returns the number of elements in this deque. 
	    */  
	   public int size() {  
	       return (tail - head) & (elements.length - 1);  
	   }  
	     /** 
	     * Returns an array containing all of the elements in this deque 
	     * in proper sequence (from first to last element). 
	     */  
	    public Object[] toArray() {  
	        return copyElements(new Object[size()]);  
    }  
	
	public void clear() {//清空操作    
	    int h = head;    
	    int t = tail;    
    if (h != t) { // head != tail 表示队列元素 不为空    
        head = tail = 0;//设置head 和 tail 初始状态    
	        int i = h;    
	        int mask = elements.length - 1;    
	        do {    
	            elements[i] = null;//配合循环将所有元素设置为null    
            i = (i + 1) & mask;    
	        } while (i != t);    
	    }    
	}    
	//遍历集合，正向遍历,从head -- tail
	public Iterator<E> iterator() {  
	       return new DeqIterator();  
	   }  
	//反向遍历
	public Iterator<E> descendingIterator() {  
	       return new DescendingIterator();  
	   }  
	public boolean contains(Object o) {//判断队列是否包含该元素    
	    if (o == null)    
	        return false;    
	    int mask = elements.length - 1;    
	    int i = head;    
	    E x;    
	    while ( (x = elements[i]) != null) {//从head元素向后猪哥判断,是否equals    
	        if (o.equals(x))    
	            return true;    
	        i = (i + 1) & mask;    
    }    
	    return false;    
	}    
	  public int size() {//获取队列元素个数,(tail - head) & (elements.length - 1)保证大小在有效范围内。    
	        return (tail - head) & (elements.length - 1);    
	    }    
	    
	  public boolean isEmpty() {//入队操作,tail+=1;出队操作head+=1;当一直出队元素的时候,head一直+，会==tail,此时head==tail都指向null元素。    
	        return head == tail;    
	    }    
```

