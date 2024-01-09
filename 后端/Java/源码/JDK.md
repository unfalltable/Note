---
title: JDK源码
categories: 源码
tags: [源码]
---

## Unsafe

简介

- 因为使Java拥有了类似c语言一样的指针，所以不安全Unsafe

- Unsafe不能直接调用

  - 通过反射获得

    ```java
    Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
    theUnsafe.setAccessible(true);
    Unsafe unsafe = theUnsafe.get(null);
    ```

  - Unsafe提供了一个静态方法getUnsafe可以获取Unsafe实例，但是必须使用引导类加载器加载Unsafe类才能使用，因为有@CallerSensitive注解标注，为危险方法

    - 命令行中输入：`java -Xbootclasspath/a: ${path}   // 其中path为调用Unsafe相关方法的类所在jar包路径 `

操作内存

- 主要包含堆外内存的分配、拷贝、释放、给定地址值操作等方法
- 为什么要使用堆外内存
  - 对垃圾回收停顿的改善。由于堆外内存是直接受操作系统管理而不是JVM，所以当我们使用堆外内存时，即可保持较小的堆内内存规模。从而在GC时减少回收停顿对于应用的影响。
  - 提升程序I/O操作的性能。通常在I/O通信过程中，会存在堆内内存到堆外内存的数据拷贝操作，对于需要频繁进行内存间数据拷贝且生命周期较短的暂存数据，都建议存储到堆外内存。
- 应用：
  - DirectByteBuffer，在NIO框架中通常作为缓冲池使用，内部的内存分配逻辑都由Unsafe提供
    - `unsafe.allocateMemory(内存大小)`
      - 分配内存，返回基地址
    - `unsafe.setMemory(base, size, (byte) 0)`
      - 设置内存，内存初始化
    - `cleaner = Cleaner.create(this, new Deallocator(base, size, cap))`
      - 跟踪DirectByteBuffer对象的垃圾回收，以实现当DirectByteBuffer被垃圾回收时，分配的堆外内存一起被释放
      - Cleaner继承自虚引用，当DirectByteBuffer对象仅被Cleaner对象引用时，
      - gc时会将DirectByteBuffer对象回收，Cleaner对象会调用clean() 方法来进行堆外内存的释放。

CAS（CompareAndSwap）

- 是一个本地方法
- AtomicInteger 使用 unsafe提供的CAS
- Unsafe只提供了 compareAndSwapObject、compareAndSwapInt、compareAndSwapLong 方法
- 赋值阶段源码中都是通过do-while判断
- 原理：
  - 通过循环判断预期值来决定是否更新值
  - 如果是多处理器，需要添加lock前缀（Lock#信号），内存屏障
  - 这里的lock前缀就是使用了处理器的总线锁(最新的处理器都使用缓存锁代替总线锁来提高性能)

    - 总线锁：将cpu和内存之间的通信锁住，其他cpu不能操作内存中的数据，开销比较大，相当于加了悲观锁
    - 缓存锁：使用缓存锁定，将原子操作放在cpu缓存中进行

      - 缓存锁定：
        - 发生共享内存的锁定时，不会锁内存，而是修改内存地址，通过**MESI缓存一致性**保证原子性
      - 缓存一致性：
        - 如果这块内存被两个以上的处理器缓存，会阻止这块内存区域的同时修改
        - 当处理器对缓存中的数据修改后，会通知其他处理器删除其内部这块内存的缓存，或者从主内存中重新读取
      - 如果操作的数据跨多个缓存行时，处理器会使用总线锁

线程调度

- 线程挂起、恢复、锁机制
- park、unpark、monitorEnter、monitorExit、tryMonitorEnter

Class相关

- 提供Class和它的静态字段的操作相关方法，包含静态字段内存定位、定义类、定义匿名类、检验&确保初始化等
- 获取属性在内存中的位置
- 获取数组第一个元素在内存中的偏移地址，还可以获取下标的增量地址

系统相关

- addressSize：返回系统指针的大小。返回值为4（32位系统）或 8（64位系统）
- pageSize：内存页的大小，此值为2的幂次方

内存屏障

- loadFence：禁止load操作重排序
- storeFence：禁止store操作重排序
- fullFence：禁止load、store操作重排序

对象操作

- 主要包含对象成员属性相关操作及非常规的对象实例化方式等相关方法
- Unsafe的allocateInstance实现对象的实例化，保证在目标类无默认构造函数时，反序列化不够影响

### getUnsafe()

```java
@CallerSensitive
public static Unsafe getUnsafe() {
    Class var0 = Reflection.getCallerClass(); //getCallerClass方法需要启动类加载的类才可以调用
    if (!VM.isSystemDomainLoader(var0.getClassLoader())) {
        throw new SecurityException("Unsafe");
    } else {
        return theUnsafe;
    }
}
```

### getAndAddInt()

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2); //检验
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

### 核心方法

```java
 //重新分配内存
 public native long reallocateMemory(long address, long bytes);  
   
 //分配内存  
 public native long allocateMemory(long bytes);  
   
 //释放内存  
 public native void freeMemory(long address);  
   
 //在给定的内存块中设置值  
 public native void setMemory(Object o, long offset, long bytes, byte value);  
   
 //从一个内存块拷贝到另一个内存块  
 public native void copyMemory(Object srcBase, long srcOffset, Object destBase, long destOffset, long bytes);  
   
 //获取值，不管java的访问限制，其他有类似的getInt，getDouble，getLong，getChar等等  
 public native Object getObject(Object o, long offset);  
   
 //设置值，不管java的访问限制，其他有类似的putInt,putDouble，putLong，putChar等等  
 public native void putObject(Object o, long offset);  
   
 //从一个给定的内存地址获取本地指针，如果不是allocateMemory方法的，结果将不确定  
 public native long getAddress(long address);  
   
 //存储一个本地指针到一个给定的内存地址,如果地址不是allocateMemory方法的，结果将不确定  
 public native void putAddress(long address, long x);  
   
 //该方法返回给定field的内存地址偏移量，这个值对于给定的filed是唯一的且是固定不变的  
 public native long staticFieldOffset(Field f);  
   
 //报告一个给定的字段的位置，不管这个字段是private，public还是保护类型，和staticFieldBase结合使用  
 public native long objectFieldOffset(Field f);  
   
 //获取一个给定字段的位置  
 public native Object staticFieldBase(Field f);  
   
 //确保给定class被初始化，这往往需要结合基类的静态域（field）  
 public native void ensureClassInitialized(Class c);  
   
 //可以获取数组第一个元素的偏移地址  
 public native int arrayBaseOffset(Class arrayClass);  
   
 //可以获取数组的转换因子，也就是数组中元素的增量地址。将arrayBaseOffset与arrayIndexScale配合使用， 可以定位数组中每个元素在内存中的位置  
 public native int arrayIndexScale(Class arrayClass);  
   
 //获取本机内存的页数，这个值永远都是2的幂次方  
 public native int pageSize();  
   
 //告诉虚拟机定义了一个没有安全检查的类，默认情况下这个类加载器和保护域来着调用者类  
 public native Class defineClass(String name, byte[] b, int off, int len, ClassLoader loader, ProtectionDomain protectionDomain);  
   
 //定义一个类，但是不让它知道类加载器和系统字典  
 public native Class defineAnonymousClass(Class hostClass, byte[] data, Object[] cpPatches);  
   
 //锁定对象，必须是没有被锁的
 public native void monitorEnter(Object o);  
   
 //解锁对象  
 public native void monitorExit(Object o);  
   
 //试图锁定对象，返回true或false是否锁定成功，如果锁定，必须用monitorExit解锁  
 public native boolean tryMonitorEnter(Object o);  
   
 //引发异常，没有通知  
 public native void throwException(Throwable ee);  
   
 //CAS，如果对象偏移量上的值=期待值，更新为x,返回true.否则false.类似的有compareAndSwapInt,compareAndSwapLong,compareAndSwapBoolean,compareAndSwapChar等等。  
 public final native boolean compareAndSwapObject(Object o, long offset,  Object expected, Object x);  
   
 // 该方法获取对象中offset偏移地址对应的整型field的值,支持volatile load语义。类似的方法有getIntVolatile，getBooleanVolatile等等  
 public native Object getObjectVolatile(Object o, long offset);   
   
 //线程调用该方法，线程将一直阻塞直到超时，或者是中断条件出现。  
 public native void park(boolean isAbsolute, long time);  
   
 //终止挂起的线程，恢复正常.java.util.concurrent包中挂起操作都是在LockSupport类实现的，也正是使用这两个方法
 public native void unpark(Object thread);  
   
 //获取系统在不同时间系统的负载情况  
 public native int getLoadAverage(double[] loadavg, int nelems);  
   
 //创建一个类的实例，不需要调用它的构造函数、初使化代码、各种JVM安全检查以及其它的一些底层的东西。即使构造函数是私有，我们也可以通过这个方法创建它的实例,对于单例模式，简直是噩梦。 
 public native Object allocateInstance(Class cls) throws InstantiationException;  
```

## 集合

### ArrayList

- 扩容

  - 使用无参构造方法创建时默认初始容量是0，使用add方法添加元素就触发扩容
  - 第一次扩容容量变为10，当存满了会再触发扩容，默认是1.5倍
  - 扩容是创建一个新数组，再将原数组内容拷贝到新数组
  - 扩容后容量 = 扩容前容量 > 1 +扩容前容量 (即1.5倍)
  - 因为每次扩容都比较小，所以要尽量避免频繁的扩容，当预知要保存多少元素时，就在创建时指定ArrayList的初始大小，或者使用ensureCapacity手动扩容

  ```java
  //数组最大值 = 整型最大值 - 8
  private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
  
  public void ensureCapacity(int minCapacity) {
      int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
          ? 0 : DEFAULT_CAPACITY;
      if (minCapacity > minExpand) {
          ensureExplicitCapacity(minCapacity);
      }
  }
  private void ensureExplicitCapacity(int minCapacity) {
      modCount++;
      if (minCapacity - elementData.length > 0)
          grow(minCapacity);
  }
  private void grow(int minCapacity) {
      int oldCapacity = elementData.length;
      int newCapacity = oldCapacity + (oldCapacity >> 1); //1.5倍
      if (newCapacity - minCapacity < 0)
          newCapacity = minCapacity;
      if (newCapacity - MAX_ARRAY_SIZE > 0)
          newCapacity = hugeCapacity(minCapacity);
      elementData = Arrays.copyOf(elementData, newCapacity);
  }
  //得到最大的数组容量
  private static int hugeCapacity(int minCapacity) {
      if (minCapacity < 0) // 太大超过int最大值就变成负数了
          throw new OutOfMemoryError();
      return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
  }
  ```


- add(E e)、addAll(Collection<? extends E> c)

  - 检查剩余空间是否需要扩容，modCount++
  - 添加到数组末尾
  - 更新size

- add(int index, E element)、addAll(int index, Collection<? extends E> c)

  - 检查下标是否越界
  - 检查剩余空间是否需要扩容，modCount++
  - 原数组的值复制到新数组，插入区间前后的元素复制到新数组不包含插入区间的位置
  - 然后在插入区间插入新增的元素
  - 更新size

- set(int index, E element)

  - 检查下标是否越界
  - 保存老值，新值插入index位置
  - 返回老值

- get(int index)

  - 检查下标是否越界
  - 返回指定下标的值，需要注意类型转换

- remove(int index)

  - 检查下标是否越界，modCount++
  - 保存老值
  - 数组从index下标之后的元素都往前移（通过复制）
  - 为了让GC起作用，最后一个位置会显式赋`null`值
  - 返回老值

- remove(Object o)

  - 删除第一个满足`o.equals(elementData[index])`的元素

- trimToSize() 将底层数组的容量调整为当前列表保存的实际元素的大小

  - modCount++
  - 判断 size 是否 小于数组大小

    - 如果size == 0 则为空数组
    - size != 0 则 将数组复制到一个size = 数组长度的新数组

- indexOf(Object o)、lastIndexOf(Object o)

  - 判断 o 是否为 null
  - 遍历数组找下标

    - o 为 null 则用 == 判断
    - o 不为 null 则用 equals 判断

  - 找到返回下标，找不到返回-1

- Fail-Fast机制

  - 通过记录modCount参数，迭代器遍历的时候遇到并发的修改时会报错

### LinkedList

- getFirst() 、getLast()
  - 定义个一个常量来接收 first 或者 last
  - 如果指向 null，抛`NoSuchElementException`异常
  - 有值就返回

- get(int index)
  - 检查下标是否合法，是否越界
  - 返回 node(index).item
- set(int index, E element)
  - 判断下标是否合法，是否越界
  - Node x = node(index) 
  - E oldVal = x.item
  - x.item = element
  - return  oldVal 
- remove(Object o)
  - 判断 o 是否为 null
  - 遍历链表找值符合的节点

    - o 为 null 则用 == 判断
    - o 不为 null 则用 equals 判断
  - 找到则用 unlink() 删除节点，成功返回true

- remove(int index)
  - 检查下标是否越界
  - 用 unlink( node(index) ) 删除节点

- removeFirst()、removeLast()
  - 定义一个常来接收 first / last
  - first / last 为null 抛NoSuchElementException异常
  - 不为空则调用 unlinkFirst() / unlinkLast() 

- unlink(Node\<E> x)
  - 定义三个常量保存节点的值 element、前驱节点 prev、后继节点 next
  - 先处理前驱，在处理后继，注意防止空指针异常
    - 判断 prev 是否为null
      - 为 null 说明是第一个节点要删除，直接 first = next
      - 不为 null，则 prev.next = next，x.prev = null
    - 判断 next 是否为null
      - 为 null 则说明是最后一个节点要删除，直接 last = prev
      - 不为 null，则 next.prev = prev，x.next = null
  - 将x的值置为 null。更新 size，modCount++
  - 返回被删除的值 element
- unlinkFirst(Node\<E> f)
  - 定义两个常量保存节点的值 element，后继节点 next
  - 将 f 的值和后继节点置为 null
  - 然后 first = next
  - 判断 next 是否等于 null
    - 为 null，则 last = null
    - 不为 null，则 next.prev = null

  - size--
  - modCount++
  - 返回被删除的值 element

- unlinkLast(Node\<E> l)
  - 定义两个常量保存节点的值 element，前驱节点 prev
  - 将 l 的值和前驱节点置为 null
  - 然后 last = prev
  - 判断 prev 是否等于 null
    - 为 null，则 first = null
    - 不为 null，则 prev.next = null
  - size--
  - modCount++
  - 返回被删除的值 element
- add(E e)
  - 调用 linkLast(e)
  - 返回 true
- add(int index, E element)
  - 判断下标是否合法，是否越界
  - 如果 下标 = size，说明是在链表最后插入，调用 linkLast(element)
  - 下标 != size，则调用 linkBefore(element, node(index))
- linkLast(E e)
  - 定义两个常量保存 尾节点 l，新节点 newNode（根据 e 创建）
  - last = newNode
  - 判断 l 是否为 null
    - 为 null，说明链表中无节点，所以 first = newNode
    - 不为 null，说明链表中有节点，所以 l.next = newNode
  - size++
  - modCount++
- node(int index) 
  - 判断 index 在链表前半段还是后半段，通过比较 size >> 1
    - 前半段：从前往后遍历找
      - x = first
      - for x = x.next
    - 后半段：从后往前遍历找
      - x = last
      - for x = x.prev

- addAll(Collection<? extends E> c)

  - 调用addAll(size, c) 实现

- addAll(int index, Collection<? extends E> c)

  - 判断下标是否合法
  - 将集合c转化为Object数组 a
  - 得到 a的长度 numNew，判断是否等于0，等于0返回false
  - 创建两个节点指针 pred、succ
  - 判断 index 是否等于 size
    - 等于，相当于在链表最后添加，则 succ = null，pred = last
    - 不等于，则 succ = node(index)，pred = succ.prev

  - 遍历 for (Object o : a) 
    - E e = (E) o      强制类型转换
    - 根据值和前驱 pred 创建节点 newNode 
    - 判断 pred 是否为 null
      - 为 null，则 first = newNode
      - 不为 null，pred.next = newNode
    - pred = newNode
  - 判断 succ 是否为 null
    - 为 null，则 last = pred
    - 不为 null，则 pred.next = succ，succ.prev =  pred 
  - size += numNew
  - modCount++
  - return true

- clear()

  - 遍历
    - 先取得当前节点的next节点
    - 然后当前节点的所有属性置为 null
    - x = next
  - first = last = null
  - size = 0
  - modCount++ 

- indexOf(Object o)

  - 定义一个index = 0
  - 判断 o 是否为 null 
    - 为 null，遍历时走 == 判断
    - 不为 null，遍历时走 equals判断
  - 找到返回index，找不到返回 -1

- lastIndexOf(Object o)

  - 定义一个index = size
  - 从后往前遍历，也是先判断 o 是否为 null 走不同的判断逻辑
  - 找到返回index，找不到返回 -1

- peek()、peekFirst()、peekLast() 

  - 定义一个常量 f 接收 first
  - 如果 f 为 null 返回 null，不为 null 返回 f.item

- poll()、pollFirst()、pollLast()

  - 定义一个常量 f 接收 first
  - 如果 f 为 null 返回 null，不为 null 返回 unlinkFirst(f) 

- removeLastOccurrence(Object o)

  - 判断 o 是否为 null 
    - 为 null，遍历时走 == 判断
    - 不为 null，遍历时走 equals判断

  - 找到用 unlink(x) 并返回true，找不到返回 false

- element()

  - 调用 getFirst()

- remove()

  - 调用 removeFirst()

- offer(E e)

  - add(e)

- offerLast(E e)

  - 调用 addLast(e)
  - 返回 true

- push(E e)

  - 调用 addFirst(e)

- pop()

  - 调用 removeFirst()

- removeFirstOccurrence(Object o)

  - 调用 remove(o)

### ArrayDeque

- addFirst(E e)
  - 判断 e 是否为null，为 null 就抛异常
  - 就是在 head 之前插入，插入后head-1
    - elements [ head = (head - 1) & (elements.length - 1) ] = e
  - 判断是否需要扩容， 如果 head == tail 就需要扩容
- addLast(E e)
  - 判断 e 是否为null，为 null 就抛异常
  - 就是在 tail 的位置插入，插入后 tail+1
    - elements[tail] = e
  - 判断下标是否越界，越界就要扩容
    - ( tail = ( tail + 1 ) & ( elements.length - 1) ) == head，为 true 就要扩容
- doubleCapacity()
  - 只有在 head == tail 的时候才需要扩容，源码中用 assert 判断，不满足条件则抛AssertionError异常
  - 保存 head 的值（下标） p，数组的长度 n
  - 计算 head 右边元素的个数 r = n - p
  - 计算扩容后的数组大小 newCapacity = n << 1，如果 newCapacity < 0 则抛IllegalStateException异常 （就是太大了，超出in最大值就会变负数）
  - 根据 newCapacity 创建一个新数组
  - 将旧数组的值复制到新数组 a，分两步复制
    - 先复制 head 右边的数据到新数组
    - 再复制 head 左边的数据到新数组
  - 将 elements 替换为新数组 a
  - head = 0，tail = n
- pollFirst()
  - 取得 head 位置的值 result
  - 判断是否为 null，为 null 则说明数组为 null ，直接返回 null
  - 将 head 位置的值置为 null
  - 更新 head
    - head = (head + 1) & (elements.length - 1)
  - 返回 result
- pollLast()
  - 获得 tial 前一位的值 t
    - t = (tail - 1) & (elements.length - 1)
  - 取得 t 位置上的值 result
  - 判断  result  是否为 null，为 null 则说明数组为 null ，直接返回 null
  - 将 t 位置上的值置为 null
  - 更新 tail
    - tail = t
  - 返回 result
- peekFirst()
  - 返回 elements[head] 的值
- peekLast()
  - 返回 elements[(tail - 1) & (elements.length - 1)] 的值

### PriorityQueue

- add(E e)、offer(E e)

  ```java
  public boolean offer(E e) {
      if (e == null)//不允许放入null元素
          throw new NullPointerException();
      modCount++;
      int i = size;
      if (i >= queue.length)
          grow(i + 1);//自动扩容
      size = i + 1;
      if (i == 0)//队列原来为空，这是插入的第一个元素
          queue[0] = e;
      else
          siftUp(i, e);//调整
      return true;
  }
  ```

- siftUp(int k, E x)

  ```java
  private void siftUp(int k, E x) {
      while (k > 0) {
          int parent = (k - 1) >>> 1;//parentNo = (nodeNo-1)/2
          Object e = queue[parent];//取得父节点的值e
          if (comparator.compare(x, (E) e) >= 0)//调用比较器的比较方法
              break;// x 大于 e 则不用交换了，直接退出
          queue[k] = e;
          k = parent;
      }
      queue[k] = x;
  }
  ```

- element()、peek()

  - 如果 size =  0，返回 null
  - 返回 (E) queue[ 0 ]

- remove()、poll()

  ```java
  public E poll() {
      if (size == 0)
          return null;
      int s = --size;
      modCount++;
      E result = (E) queue[0];//0下标处的那个元素就是最小的那个
      E x = (E) queue[s];
      queue[s] = null;
      if (s != 0)
          siftDown(0, x);//调整
      return result;
  }
  ```

- siftDown(int k, E x)

  - 得到 size 的一半 half
  - 循环 k < half 

    - 得到左孩子的下标  child = (k << 1) + 1
    - 得到左孩子下标对应的值 c
    - 得到右孩子的下标 right = child + 1
    - 如果 right < size 并且 c > queue[right]，则 c = queue[child = right]，就是找左右孩子小的那一个
    - 如果 x < c 退出循环
    - queue[k] = c，因为 x > c
    - 交换后从 k = child
  - queue[ k ] = x

  ```java
  private void siftDown(int k, E x) {
      int half = size >>> 1;
      while (k < half) {
      	//首先找到左右孩子中较小的那个，记录到c里，并用child记录其下标
          int child = (k << 1) + 1;//leftNo = parentNo*2+1
          Object c = queue[child];
          int right = child + 1;
          if (right < size &&
              comparator.compare((E) c, (E) queue[right]) > 0)
              c = queue[child = right];
          if (comparator.compare(x, (E) c) <= 0)
              break;
          queue[k] = c;//然后用c取代原来的值
          k = child;
      }
      queue[k] = x;
  }
  ```

- remove(Object o)

  - 遍历数组找到符合值的下标 i = indexOf(o)

  - 如果 i = -1 ，返回 false

  - 得到数组最后一个元素的下标 s = --size

  - 如果 s == i，说明要删除的是最后一个元素，则 queue[i] = null

  - s != i

    - 取出数组最后一个元素的值 moved
    - queue[ s ] = null
    - siftDown(i, moved)

  - 返回 true

  ```java
  public boolean remove(Object o) {
  	//通过遍历数组的方式找到第一个满足o.equals(queue[i])元素的下标
      int i = indexOf(o);
      if (i == -1)
          return false;
      int s = --size;
      if (s == i) //情况1
          queue[i] = null;
      else {
          E moved = (E) queue[s];
          queue[s] = null;
          siftDown(i, moved);//情况2
          ......
      }
      return true;
  }
  ```

### Vector



### HashSet

### LinkedHashMap

#### 源码

- get() 与hashmap的get方法几乎一致

- addEntry(int hash, K key, V value, int bucketIndex)

  ```java
  void addEntry(int hash, K key, V value, int bucketIndex) {
      if ((size >= threshold) && (null != table[bucketIndex])) {
          resize(2 * table.length);// 自动扩容，并重新哈希
          hash = (null != key) ? hash(key) : 0;
          bucketIndex = hash & (table.length-1);// hash%table.length
      }
      // 1.在冲突链表头部插入新的entry
      HashMap.Entry<K,V> old = table[bucketIndex];
      Entry<K,V> e = new Entry<>(hash, key, value, old);
      table[bucketIndex] = e;
      // 2.在双向链表的尾部插入新的entry
      e.addBefore(header);
      size++;
  }
  ```

  ```java
  private void addBefore(Entry<K,V> existingEntry) {
      after  = existingEntry;
      before = existingEntry.before;
      before.after = this;
      after.before = this;
  }
  ```

- removeEntryForKey(Object key)

  ```java
  final Entry<K,V> removeEntryForKey(Object key) {
  	......
  	int hash = (key == null) ? 0 : hash(key);
      int i = indexFor(hash, table.length);// hash&(table.length-1)
      Entry<K,V> prev = table[i];// 得到冲突链表
      Entry<K,V> e = prev;
      while (e != null) {// 遍历冲突链表
          Entry<K,V> next = e.next;
          Object k;
          if (e.hash == hash &&
              ((k = e.key) == key || (key != null && key.equals(k)))) {// 找到要删除的entry
              modCount++; size--;
              // 1. 将e从对应bucket的冲突链表中删除
              if (prev == e) table[i] = next;
              else prev.next = next;
              // 2. 将e从双向链表中删除
              e.before.after = e.after;
              e.after.before = e.before;
              return e;
          }
          prev = e; e = next;
      }
      return e;
  }
  ```

### HashMap

#### 源码 1.8

- put( K key, V value )

  ```java
  public V put(K key, V value) {
      return putVal(hash(key), key, value, false, true);
  }
  ```

- putVal( int hash, K key, V value, boolean onlyIfAbsent, boolean evict )

  - 判断 p 是否和要插入的 key 相等，这个 p 是这个位置上的第一个元素

    - 相等则用 e 接收这个节点（保留）
    - 不相等判断 p 是不是树节点
      - 是树节点，调用红黑树的插入 ( ( TreeNode<K,V> ) p ).putTreeVal( this, tab, hash, key, value )，并用 e 接收返回值
      - 是链表节点
        - 遍历链表节点
          - 用 e 接收 p.next，判断 e 是否为null
            - 为 null 直接插入p.next
            - 如果链表长度大于等于8，则需要转化为红黑树，调用 treeifyBin(tab, hash)，跳出循环
          - 如果在该链表中找到了相等的 key (== 或 equals)，跳出循环
          - p = e
    - 如果 e != null，说明存在旧值的key与要插入的key相等
      - 取得 e.value 为 oldValue
      - 覆盖旧值 e.value = value
      - 返回旧值 oldValue

  - ++modCount

  - 如果因为插值 size > 扩容阈值，就执行扩容 resize()

  - 返回 null

  ```java
  final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
      //定义一个 tab 接收数组，p 接收下标的第一个元素，n 接收数组长度，i 接收下标
      Node<K,V>[] tab; Node<K,V> p; int n, i;
      // 第一次 put 值的时候，会触发扩容，0 -> 16
      if ((tab = table) == null || (n = tab.length) == 0)  n = (tab = resize()).length;
      // pqu如果p为 null，说明该位置还没有值，直接插入即可
      if ((p = tab[i = (n - 1) & hash]) == null) tab[i] = newNode(hash, key, value, null);
      // 数组该位置有数据
      else {
          //局部变量e储存插入的位置，k储存p的
          Node<K,V> e; K k;
          // 首先，判断该位置的第一个数据和我们要插入的数据，key 是不是"相等"，如果是，取出这个节点
          if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
              e = p;
          // 如果该节点是代表红黑树的节点，调用红黑树的插值方法，本文不展开说红黑树
          else if (p instanceof TreeNode)
              e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
          else {
              // 到这里，说明数组该位置上是一个链表
              for (int binCount = 0; ; ++binCount) {
                  // 插入到链表的最后面(Java7 是插入到链表的最前面)
                  if ((e = p.next) == null) {
                      p.next = newNode(hash, key, value, null);
                      // TREEIFY_THRESHOLD 为 8，所以，如果新插入的值是链表中的第 8 个
                      // 会触发下面的 treeifyBin，也就是将链表转换为红黑树
                      if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                          treeifyBin(tab, hash);
                      break;
                  }
                  // 如果在该链表中找到了"相等"的 key(== 或 equals)
                  if (e.hash == hash &&
                      ((k = e.key) == key || (key != null && key.equals(k))))
                      // 此时 break，那么 e 为链表中[与要插入的新值的 key "相等"]的 node
                      break;
                  p = e;
              }
          }
          // e!=null 说明存在旧值的key与要插入的key"相等"
          // 对于我们分析的put操作，下面这个 if 其实就是进行 "值覆盖"，然后返回旧值
          if (e != null) {
              V oldValue = e.value;
              if (!onlyIfAbsent || oldValue == null)
                  e.value = value;
              afterNodeAccess(e);
              return oldValue;
          }
      }
      ++modCount;
      // 如果 HashMap 由于新插入这个值导致 size 已经超过了阈值，需要进行扩容
      if (++size > threshold)
          resize();
      afterNodeInsertion(evict);
      return null;
  }
  ```

- resize()

  - 定义一个节点数组保存原始数据 oldTab

  - 定义一个 oldCap 保存旧数组容量，oldThr 保存旧阈值，定义 newCap, newThr 为 0

  ```java
  if (oldCap > 0) { // 对应数组扩容
      if (oldCap >= MAXIMUM_CAPACITY) {
          threshold = Integer.MAX_VALUE;
          return oldTab;
      }
      // 将数组大小扩大一倍
      else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
          // 将阈值扩大一倍
          newThr = oldThr << 1; 
  } else if (oldThr > 0) // 使用指定容量的构造函数创建时，因为没put所以 oldCap<=0
      newCap = oldThr;
  else {// 使用无参构造函数创建并第一次put的时候
      newCap = DEFAULT_INITIAL_CAPACITY;
      newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
  }
  //如果走了上面第一个if 中的 if 和 第一个 if 的else if 时
  //newThr是没有值的，所以需要赋值，需要判断是哪一种情况
  if (newThr == 0) {
      float ft = (float)newCap * loadFactor;
      newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                (int)ft : Integer.MAX_VALUE);
  }
  threshold = newThr;
  ```

  - 创建新数组并转移数据

    ```java
    //用新的数组大小初始化新的数组
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab; // 如果是初始化数组，到这里就结束了，返回 newTab 即可
    // 开始遍历原数组，进行数据迁移。
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                // 如果该数组位置上只有单个元素，那就简单了，简单迁移这个元素就可以了
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                // 如果是红黑树，具体我们就不展开了
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { 
                    // 这块是处理链表的情况，
                    // 需要将此链表拆成两个链表，放到新的数组中，并且保留原来的先后顺序
                    // loHead、loTail 对应一条链表，hiHead、hiTail 对应另一条链表，代码还是比较简单的
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } 
                    while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        // 第一条链表
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        // 第二条链表的新的位置是 j + oldCap，这个很好理解
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
    ```

- get(Object key)

  ```java
  return getNode(hash(key), key) == null ? null : e.value
  ```

- getNode(int hash, Object key)

  ```java
  final Node<K,V> getNode(int hash, Object key) {
      Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
      //表不为空，长度大于0，下标处有值
      if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) {
          // 判断第一个节点是不是就是需要的
          if (first.hash == hash &&  ((k = first.key) == key || (key != null && key.equals(k))))
              return first;
          //大于一个节点
          if ((e = first.next) != null) {
              // 判断是否是红黑树
              if (first instanceof TreeNode)
                  return ((TreeNode<K,V>)first).getTreeNode(hash, key);
              // 链表遍历
              do {
                  if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                      return e;
              }while ((e = e.next) != null);
          }
      }
      return null;
  }
  ```

### TreeMap

#### 源码

- 左旋

  ```java
  /*
  		p                         pr(r)
  	   / \                        / \
  	 pl   pr(r)       ==>        p   rr
  	      / \                   / \
  	    rl   rr               pl  rl
  	
  */
  private void rotateLeft(Entry<K,V> p) {
      if (p != null) {
          Entry<K,V> r = p.right;
          p.right = r.left;
          if (r.left != null)
              r.left.parent = p;
          r.parent = p.parent;
          if (p.parent == null)
              root = r;
          else if (p.parent.left == p)
              p.parent.left = r;
          else
              p.parent.right = r;
          r.left = p;
          p.parent = r;
      }
  }
  ```

- 右旋

  ```java
  /*
  		p                         pl(l)
  	   /  \                        / \
     (l)pl   pr       ==>          ll   p
  	/ \                              / \
    ll   lr                           lr  pr
  	
  */
  private void rotateRight(Entry<K,V> p) {
      if (p != null) {
          Entry<K,V> l = p.left;
          p.left = l.right;
          if (l.right != null) 
              l.right.parent = p;
          l.parent = p.parent;
          if (p.parent == null)
              root = l;
          else if (p.parent.right == p)
              p.parent.right = l;
          else p.parent.left = l;
          l.right = p;
          p.parent = l;
      }
  }
  ```

- 寻找节点后继

  ```JAVA
  static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
      if (t == null){
          return null;
      }
      // 1. t的右子树不空，则t的后继是其右子树中最小的那个元素，即最左的元素
      else if (t.right != null) {
          Entry<K,V> p = t.right;
          while (p.left != null)
              p = p.left;
          return p;
      }
      // 2. t的右孩子为空，则t的后继是其第一个向左走的祖先，即parent.right = t的
      else {
          Entry<K,V> p = t.parent;
          Entry<K,V> ch = t;
          while (p != null && ch == p.right) {
              ch = p;
              p = p.parent;
          }
          return p;
      }
  }
  ```

- get

  ```java
  final Entry<K,V> getEntry(Object key) {
      ......
      if (key == null)//不允许key值为null
          throw new NullPointerException();
      Comparable<? super K> k = (Comparable<? super K>) key;//使用元素的自然顺序
      Entry<K,V> p = root;
      while (p != null) {
          int cmp = k.compareTo(p.key);
          if (cmp < 0)//向左找
              p = p.left;
          else if (cmp > 0)//向右找
              p = p.right;
          else
              return p;
      }
      return null;
  }
  ```

- put()

  ```java
  public V put(K key, V value) {
  	......
      Entry<K,V> t = this.root;
      //如果根节点为null，当前节点直接作为根节点
      if(t == null) this.root = new Entry<>(key, value, parent);
      int cmp;
      Entry<K,V> parent; //定义一个父亲节点
      if (key == null)  throw new NullPointerException();
      Comparable<? super K> k = (Comparable<? super K>) key;//使用元素的自然顺序
      do {
          parent = t;
          cmp = k.compareTo(t.key); //与当前值比较大小
          if (cmp < 0) t = t.left;//向左找 插入的值小于当前值
          else if (cmp > 0) t = t.right;//向右找 插入的值大于当前值
          else return t.setValue(value); //当前值等于插入值
      } while (t != null); //非叶子节点
      //到达叶子节点
      Entry<K,V> e = new Entry<>(key, value, parent);//创建并插入新的entry
      if (cmp < 0) parent.left = e;
      else parent.right = e;
      fixAfterInsertion(e);//调整（旋转和变色）
      size++;
      return null;
  }
  ```

  ```java
  //调整插入的情况
  /*         左三               右三                     3树左                     3树右
  			a（黑）         a（黑）                    a（黑）                  a（黑）
  	   	   /                 \                       /   \                     /   \
  	 	 b（红）               b（红）            b（红） c（红）            b（红）  c（红）
      	/                       \                 /                                   \
     	   x（红）                    x（红）        x（红）                                x（红）
  */
  private void fixAfterInsertion(Entry<K,V> x) {
      x.color = RED;//新插入的节点都是红色的
      //父节点为红色时需要调整
      while (x != null && x != root && x.parent.color == RED) {
          //x的父节点是爷爷节点的左节点（处理左三和3树左的情况）
          if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
              //取出父亲节点的父亲节点的右节点（叔叔节点）
              Entry<K,V> y = rightOf(parentOf(parentOf(x)));
              //判断是否是3树左，如果叔叔节点为空是false，不为空则是3树左的情况
              if (colorOf(y) == RED) {
                  setColor(parentOf(x), BLACK);              //x的父亲设置为黑色
                  setColor(y, BLACK);                        //叔叔节点设置为黑色
                  setColor(parentOf(parentOf(x)), RED);      //爷爷节点设置为红色
                  x = parentOf(parentOf(x));                 //x指向爷爷节点，如果爷爷节点的父节点也是红接着处理
              } 
              //左3
              else {
                  //x是父节点的右节点（非完全左3），需要先左旋为左3，再右旋
                  if (x == rightOf(parentOf(x))) {
                      x = parentOf(x);                       
                      rotateLeft(x);                        //在旋转时会改回父子间的指向
                  }
                  //x是父节点的左节点
                  setColor(parentOf(x), BLACK);             //x的父亲设置为黑色
                  setColor(parentOf(parentOf(x)), RED);     //爷爷节点设置为红色
                  rotateRight(parentOf(parentOf(x)));       //右旋 
              }
          } 
          //x的父节点是爷爷节点的右节点（处理右三和3树右的情况）
          else {
              //取出父亲节点的父亲节点的左节点（叔叔节点）
              Entry<K,V> y = leftOf(parentOf(parentOf(x)));
              //判断是否是3树右，如果叔叔节点为空是false，不为空则是3树右的情况
              if (colorOf(y) == RED) {
                  setColor(parentOf(x), BLACK);              //x的父亲设置为黑色
                  setColor(y, BLACK);                        //叔叔节点设置为黑色
                  setColor(parentOf(parentOf(x)), RED);      //爷爷节点设置为红色
                  x = parentOf(parentOf(x));                 //x指向爷爷节点，如果爷爷节点的父节点也是红接着处理
              } 
              //右3
              else {
                  //x是父节点的左节点（非完全右3），需要先右旋为右3，再左旋
                  if (x == leftOf(parentOf(x))) {
                      x = parentOf(x);                       
                      rotateRight(x);                        //在旋转时会改回父子间的指向
                  }
                  setColor(parentOf(x), BLACK);              //x的父亲设置为黑色
                  setColor(parentOf(parentOf(x)), RED);      //爷爷节点设置为红色
                  rotateLeft(parentOf(parentOf(x)));         //左旋 
              }
          }
      }
      root.color = BLACK;			//根节点必须为黑色
  }
  ```

- remove()

  ```java
  /*
  	删除的情况：
  		1.删除节点是叶子节点，直接删除
  		2.删除节点有左孩子或者右孩子，删除节点后让其左孩子或右孩子指向向删除节点的父节点
  		3.删除节点有左右孩子节点，需要找到当前节点的后继节点，替代当前节点                    
  */
  private void deleteEntry(Entry<K,V> p) {
      modCount++;
      size--;
      //情况3，将其转换成情况2来处理
      if (p.left != null && p.right != null) 
          Entry<K,V> s = successor(p);
          p.key = s.key;
          p.value = s.value;
          p = s; //p指向它的后继节点
      }
  	//情况2
  	//有左孩子取左孩子，没有就取右（因为我们找的是后继节点，是不会有左孩子的，但源码为了代码健壮性）
      Entry<K,V> replacement = (p.left != null ? p.left : p.right);
  	//有右孩子
      if (replacement != null) {
          //左孩子或右孩子指向向删除节点的父节点
          replacement.parent = p.parent;
          if (p.parent == null) //父节点为空说明是根节点，直接替换
              root = replacement;
          else if (p == p.parent.left) //是父节点的左孩子，重新修改指向
              p.parent.left  = replacement;
          else						//是父节点的右孩子，重新修改指向
              p.parent.right = replacement;
          p.left = p.right = p.parent = null; //被删除的节点的引用全部置为null，gc处理
          if (p.color == BLACK)			 //如果删除节点的颜色为黑色需要进行调整
              fixAfterDeletion(replacement);// 调整
      } 
  	//无左右孩子且父节点为null，即删除的节点是根节点
  	else if (p.parent == null) {
          root = null;	      
      } 
  	//情况1
  	else {
          if (p.color == BLACK)
              fixAfterDeletion(p);// 调整
          if (p.parent != null) {
              if (p == p.parent.left)
                  p.parent.left = null;
              else if (p == p.parent.right)
                  p.parent.right = null;
              p.parent = null;
          }
      }
  }
  ```

  ```java
  /*
  	情况1：被删除的节点有左右孩子的节点，需要通过改变节点指向和变色就可以解决的
  	情况2：被删除的节点无左右孩子的节点，用父亲节点替代，再用一个兄弟节点的子节点替代父亲节点
  		情况2.1：兄弟节点有一个孩子，旋转成有右孩子的情况，在进行左旋
  		情况2.2：兄弟节点有两个孩子，直接进行左旋
  	情况3：被删除的节点无左右孩子的节点，用父亲节点替代，但是兄弟节点没有子节点可以替代父亲节点	
  */
  private void fixAfterDeletion(Entry<K,V> x) {
      //情况2，3
      while (x != root && colorOf(x) == BLACK) {
          //x是左孩子的情况
          if (x == leftOf(parentOf(x))) {
              //找到x的兄弟节点
              Entry<K,V> sib = rightOf(parentOf(x));
              //判断sib是否是x的兄弟节点，只有sib是黑色时才是兄弟节点，红色需要调整
              if (colorOf(sib) == RED) {
                  setColor(sib, BLACK);                   // 兄弟变黑
                  setColor(parentOf(x), RED);             // 兄弟父亲变红
                  rotateLeft(parentOf(x));                // 左旋，将左孩子变成右孩子
                  sib = rightOf(parentOf(x));             // 修改找到真正的兄弟节点
              }
              //情况3，兄弟节点无左右孩子节点
              if (colorOf(leftOf(sib))  == BLACK && colorOf(rightOf(sib)) == BLACK) {
                  setColor(sib, RED);                     // 设置兄弟节点为红色
                  x = parentOf(x);                        // 删除的节点指向x的父亲，递归条件
              } 
              //情况2，兄弟节点有一个或俩个子节点
              else {
                  //没有右孩子，需要右旋为有左孩子的情况
                  if (colorOf(rightOf(sib)) == BLACK) {
                      setColor(leftOf(sib), BLACK);       // 将兄弟节点的左节点变黑
                      setColor(sib, RED);                 // 将兄弟节点变红
                      rotateRight(sib);                   // 右旋
                      sib = rightOf(parentOf(x));         // 兄弟节点重新指向为父亲节点的右节点
                  }
                  setColor(sib, colorOf(parentOf(x)));    // 兄弟节点要变成父亲节点的颜色
                  setColor(parentOf(x), BLACK);           // 父亲节点变黑
                  setColor(rightOf(sib), BLACK);          // 兄弟节点的右孩子变黑
                  rotateLeft(parentOf(x));                // 左旋
                  x = root;                               // 结束循环
              }
          } 
          //x是右孩子的情况，不会出现这种可能
          else { 
              Entry<K,V> sib = leftOf(parentOf(x));
              if (colorOf(sib) == RED) {
                  setColor(sib, BLACK);                   // 
                  setColor(parentOf(x), RED);             // 
                  rotateRight(parentOf(x));               // 
                  sib = leftOf(parentOf(x));              // 
              }
              if (colorOf(rightOf(sib)) == BLACK &&
                  colorOf(leftOf(sib)) == BLACK) {
                  setColor(sib, RED);                     // 
                  x = parentOf(x);                        // 
              } else {
                  if (colorOf(leftOf(sib)) == BLACK) {
                      setColor(rightOf(sib), BLACK);      // 
                      setColor(sib, RED);                 // 
                      rotateLeft(sib);                    // 
                      sib = leftOf(parentOf(x));          // 
                  }
                  setColor(sib, colorOf(parentOf(x)));    // 
                  setColor(parentOf(x), BLACK);           // 
                  setColor(leftOf(sib), BLACK);           // 
                  rotateRight(parentOf(x));               // 
                  x = root;                               // 
              }
          }
      }
      //情况1，变黑，指向在前面已经修改完毕了
      setColor(x, BLACK);
  }
  ```

### HashTable

### WeakHashMap

## 注解

### @CallerSensitive

- 该注解标注的方法为危险方法，调用该注解标注的方法的对象必须也需要有该注解，且改对象必须有启动类加载
- 开发者自己写的@CallerSensitive不可被识别即无法调用该危险方法
- 可以通过jvm参数 `-Xbootclasspath/a: path` 伪装为启动类

### @FunctionInterface

- 接口中只有一个抽象方法，即可搭配lamda表达式使用

### @Contended

- 防止缓存行，伪共享
- 对齐填充，因为一个缓存行是64字节，当读取的目标数据小于64字节时，可以填充一些无意义的数据使其字节接近或达到64字节，这样就可以使一个缓存行只被一个线程所持有