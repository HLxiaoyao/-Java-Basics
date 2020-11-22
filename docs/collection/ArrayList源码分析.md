# ArrayList源码分析
ArrayList本质上是一个数组，它内部通过对数组的操作实现了List功能，所以ArrayList又被叫做动态数组。每个ArrayList实例都有容量并且会自动扩容。它可添加null，有序可重复，线程不安全。Vector和ArrayList内部实现基本是一致的，除了Vector添加了synchronized保证其线程安全。

![](/images/collection/ArrayList继承体系.png)

## ArrayList成员属性
```
// 默认初始容量
private static final int DEFAULT_CAPACITY = 10;

// 初始容量为0时，elementData指向此对象(空元素对象)
private static final Object[] EMPTY_ELEMENTDATA = {};

// 调用ArrayList()构造方法时，elementData指向此对象(默认容量空元素对象)
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

// 用于存储集合元素，非private类型能够简化内部类的访问(在编译阶段)
transient Object[] elementData;

// 包含的元素个数
private int size;
```
为什么DEFAULT_CAPACITY变量要声明为private static final类型呢？

    （1）private是为了把变量的作用范围控制在类中。
    （2）static修饰的变量是静态变量。JVM只会为静态变量分配一次内存，这样无论对象被创建多少次，此变量始终指向的都是同一内存地址，达到节省内存，提升性能的目的。
    （3）final修饰的变量在被初始化后，不可再被指向别的内存地址，以防变量的地址被篡改。

## ArrayList构造方法
```
// 无参构造方法
public ArrayList() {
    // 无参构造器方法，将elementData指向DEFAULTCAPACITY_EMPTY_ELEMENTDATA
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

// 指定容量构造方法  
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        // initialCapacity大于0时，将elementData指向新建的initialCapacity大小的数组。
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        // initialCapacity为空时，将elementData指向EMPTY_ELEMENTDATA
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: " +
                initialCapacity);
    }
}

// 指定集合构造方法
public ArrayList(Collection<? extends E> c) {
    // 将elementData指向c转换后的数组
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray 可能不会返回Object[]，所以需要手动检查下        
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // 如果elementData.length为0，将elementData指向EMPTY_ELEMENTDATA
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```
ArrayList的无参构造方法使用频率是非常高的，在第一次添加元素时会将capacity初始化为10。在知道ArrayList中需要存储多少元素时，使用指定容量构造方法，可避免扩容带来的运行开销，提高程序运行效率。当需要复用Collection对象时，使用指定集合构造方法。

## ArrayList插入操作
### 插入ArrayList尾部
```
// 将指定的元素添加到列表的末尾
public boolean add(E e) {
    // 确保内部容量，如果容量不够则计算出所需的容量值
    ensureCapacityInternal(size + 1);
    // 将元素插入到数组尾部，size加一
    elementData[size++] = e;
    return true;
}
```
add(E e)方法的平均时间复杂度是O(1)，它的流程大体上分为两步：

    （1）保证内部容量可用，若容量不够会进行自动扩容；
    （2）将元素添加到数组尾部；

![](/images/collection/ArrayList插入元素到尾部.png)

### 插入ArrayList指定位置
```
// 在列表的指定位置上添加指定元素，在添加之前将在此位置上的元素及其后面的元素向右移一位
public void add(int index, E element) {
    // 检查索引是否越界
    rangeCheckForAdd(index);
    // 确保内部容量，如果容量不够则计算出所需的容量值
    ensureCapacityInternal(size + 1);
    // 将index及index之后的元素向右移一位
    System.arraycopy(elementData, index, elementData, index + 1,
            size - index);
    // 将新元素插入到index处
    elementData[index] = element;
    // 元素个数加一
    size++;
}

// 检查索引是否越界
private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```
add(int index, E element)的平均时间复杂度是O(N)，所以在大容量的集合中不要频繁使用此方法，否则可能会产生效率问题。在指定位置添加元素的流程如下图所示:

![](/images/collection/ArrayList插入元素到指定位置.png)

### ArrayList自动扩容
```
// 是否需要扩容
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

// 计算容量
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    // 如果elementData指向DEFAULTCAPACITY_EMPTY_ELEMENTDATA，返回DEFAULT_CAPACITY和minCapacity中的较大值
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    // 否则直接返回minCapacity
    return minCapacity;
}

// 确保明确的容量
private void ensureExplicitCapacity(int minCapacity) {
    // 修改次数加一
    modCount++;

    // 如果minCapacity大于elementData数组长度，那么进行扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

// 要分配的最大数组大小
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
* 增加容量确保数组至少能够容纳最小容量参数指定的元素个数
* @param minCapacity 所需的最小容量
*/
private void grow(int minCapacity) {
    // 声明oldCapacity为elementData长度
    int oldCapacity = elementData.length;
    // 将newCapacity声明为oldCapacity的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 如果newCapacity小于minCapacity，将newCapacity指向minCapacity
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    // 如果newCapacity超出了最大数组长度，调用hugeCapacity()方法计算newCapacity
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 根据newCapacity生成一个新的数组，并将elementData老数据放入elementData中
    elementData = Arrays.copyOf(elementData, newCapacity);
}

/**
* @param minCapacity 最小容量
* @return 计算后的容量
*/
private static int hugeCapacity(int minCapacity) {
    // 如果minCapacity小于0，抛出内存溢出异常
    if (minCapacity < 0) 
        throw new OutOfMemoryError();
    // 比较得出所需容量
    return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
}
```
自动扩容的流程参看上面代码，总结几个要点：

    （1）如果使用的是new ArrayList()构造方法，在添加第一个元素时，容量会被设置为DEFAULT_CAPACITY(10)大小
    （2）若非第一次添加元素，容量会被扩容为之前的1.5倍。若扩容1.5倍后，还是小于用户需要的容量，则直接扩容为用户需要的容量minCapacity
    （3）若上一步扩容后的容量大于MAX_ARRAY_SIZE，那么将Integer.MAX_VALUE作为容量
    
## ArrayList删除操作
### 删除ArrayList指定位置
```
// 移除指定索引位置元素
public E remove(int index) {
    // index越界检查
    rangeCheck(index);

    modCount++;
    // 获取将要删除的元素
    E oldValue = elementData(index);
    // 获取将要移动的元素个数
    int numMoved = size - index - 1;
    if (numMoved > 0)
        // 将index之后的元素向左移一位
        System.arraycopy(elementData, index + 1, elementData, index, numMoved);
    // size减一，并将elementData数组最后一个元素指向null，让GC进行操作
    elementData[--size] = null; 
    return oldValue;
}

@SuppressWarnings("unchecked")
E elementData(int index) {
    return (E) elementData[index];
}
```
### 删除ArrayList指定元素
```
// 删除指定元素
public boolean remove(Object o) {
    // 如果对象为null，删除数组中的第一个为null的元素。没有null元素的话则不会变化
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        //删除数组中的第一个为o的元素，没有则不操作
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

// 省略了越界检查，而且不会返回被删除的值，反映了JDK将性能优化到极致
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index + 1, elementData, index,
                    numMoved);
    elementData[--size] = null;
}
```
删除操作的平均时间复杂度是O(N)，其主要步骤：

    （1）通过遍历ArrayList，中List中找到指定元素的index
    （2）将索引后面的元素向左移一位;
    （3）数组的最后一个元素赋值为null;
    （4）size减一;

![](/images/collection/ArrayList删除元素.png)

## ArrayList的fail-fast机制
fail-fast机制是java集合(Collection)中的一种错误机制。当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。例如当某一个线程A通过iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。

### fail-fast原理
产生fail-fast事件，是通过抛出ConcurrentModificationException异常来触发的。
那么ArrayList是如何抛出ConcurrentModificationException异常的呢?

ConcurrentModificationException是在操作Iterator时抛出的异常。先看看Iterator的源码。ArrayList的Iterator是在父类AbstractList.java中实现的。代码如下：
```
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
    ...

    // AbstractList中唯一的属性，用来记录List修改的次数：每修改一次(添加/删除等操作)，将modCount+1
    protected transient int modCount = 0;

    // 返回List对应迭代器。实际上是返回Itr对象。
    public Iterator<E> iterator() {
        return new Itr();
    }

    // Itr是Iterator(迭代器)的实现类
    private class Itr implements Iterator<E> {
        int cursor = 0;
        int lastRet = -1;
        // 修改数的记录值。每次新建Itr()对象时，都会保存新建该对象时对应的modCount；
        // 以后每次遍历List中的元素的时候，都会比较expectedModCount和modCount是否相等；若不相等，则抛出ConcurrentModificationException异常，产生fail-fast事件。
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size();
        }

        public E next() {
            // 获取下一个元素之前，都会判断“新建Itr对象时保存的modCount”和“当前的modCount”是否相等；
            // 若不相等，则抛出ConcurrentModificationException异常，产生fail-fast事件。
            checkForComodification();
            try {
                E next = get(cursor);
                lastRet = cursor++;
                return next;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        public void remove() {
            if (lastRet == -1)
                throw new IllegalStateException();
            checkForComodification();

            try {
                AbstractList.this.remove(lastRet);
                if (lastRet < cursor)
                    cursor--;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException e) {
                throw new ConcurrentModificationException();
            }
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
    ...
}
```
从上述代码可以看出在调用next()和remove()时，都会执行checkForComodification()。若modCount不等于expectedModCount，则抛出ConcurrentModificationException异常，产生fail-fast事件。

要搞明白fail-fast机制，需要理解什么时候modCount不等于expectedModCount。从Itr类中知道expectedModCount在创建Itr对象时，被赋值为modCount。那么modCount何时会被修改呢？
从ArrayList源码分析可以看出，无论是add()、remove()，还是clear()，只要涉及到修改集合中的元素个数时，都会改变modCount的值。

总结一下fail-fast是如何产生的？当多个线程对同一个集合进行操作的时候，某线程访问集合的过程中，该集合的内容被其他线程所改变(即其它线程通过add、remove、clear等方法，改变了modCount的值)；这时就会抛出ConcurrentModificationException异常，产生fail-fast事件。

### 解决fail-fast的原理
接下来看一下java.util.concurrent包中是如何解决fail-fast事件的？以与ArrayList对应的CopyOnWriteArrayList进行说明。先看看CopyOnWriteArrayList的源码：
```
public class CopyOnWriteArrayList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    ...

    // 返回集合对应的迭代器
    public Iterator<E> iterator() {
        return new COWIterator<E>(getArray(), 0);
    }

    ...
   
    private static class COWIterator<E> implements ListIterator<E> {
        private final Object[] snapshot;
        private int cursor;

        private COWIterator(Object[] elements, int initialCursor) {
            cursor = initialCursor;
            // 新建COWIterator时，将集合中的元素保存到一个新的拷贝数组中。
            // 这样当原始集合的数据改变，拷贝数据中的值也不会变化。
            snapshot = elements;
        }

        public boolean hasNext() {
            return cursor < snapshot.length;
        }

        public boolean hasPrevious() {
            return cursor > 0;
        }

        public E next() {
            if (! hasNext())
                throw new NoSuchElementException();
            return (E) snapshot[cursor++];
        }

        public E previous() {
            if (! hasPrevious())
                throw new NoSuchElementException();
            return (E) snapshot[--cursor];
        }

        public int nextIndex() {
            return cursor;
        }

        public int previousIndex() {
            return cursor-1;
        }

        public void remove() {
            throw new UnsupportedOperationException();
        }

        public void set(E e) {
            throw new UnsupportedOperationException();
        }

        public void add(E e) {
            throw new UnsupportedOperationException();
        }
    }
    ...
}
```
从上述源码可以看出：

    （1）与ArrayList继承于AbstractList不同，CopyOnWriteArrayList没有继承于AbstractList，它仅仅只是实现了List接口。
    （2）ArrayList的iterator()函数返回的Iterator是在AbstractList中实现的；而CopyOnWriteArrayList是自己实现Iterator。
    （3）ArrayList的Iterator实现类中调用next()时，会调用checkForComodification()比较expectedModCount和modCount的大小；但是CopyOnWriteArrayList的Iterator实现类中，没有所谓的checkForComodification()，更不会抛出ConcurrentModificationException异常。
    
fail-fast问题：

（1）在使用for循环对ArrayList进行遍历的时候，采用add操作修改ArrayList时会抛出ConcurrentModificationException异常吗？

    是不会的，因为iterator是通过容器自己实现的内部类，做迭代操作时就必须获取相应容器实现的itr对象，再通过其内部实现去迭代数据元素。因为for循环遍历时是针对底层数组下标直接进行操作，因此对于普通的for循环进行add操作并不会去检查相应的modcount是否发生了变动。
    
（2）fail-fast在单线程环境下是否会产生？

    也会产生。
    
## 线程安全
ArrayList并不是线程安全的集合，源码剖析也展示了ArrayList通过modCount以及fali-fast机制去避免一定程度的线程安全问题，那么如何保证ArrayList的线程安全呢？其实可以通过以下方案实现：

    （1）使用Vector代替ArrayList。
    （2）使用 Collections.synchronizedList包装ArrayList，然后操作包装后的list即可。
    （3）使用CopyOnWriteArrayList代替ArrayList。
    （4）在使用ArrayList时，应用程序通过同步机制去控制ArrayList的读写，不建议。
    
未解决：转化为数组、序列化