# LinkedList源码分析
LinkedList底层通过双向集合的数据结构实现

    （1）内存无需连续的空间保证
    （2）元素查找只能是顺序的遍历查找
    （3）针对增删操作具有更好的性能
    
LinkedList可以作为List使用，也可以作为队列和栈使用。支持从集合的头部、中间、尾部进行添加、删除等操作。

## LinkedList成员属性
```
// 长度
transient int size = 0;

// LinkedList头节点。
transient Node<E> first;

// LinkedList尾节点
transient Node<E> last;
```
## LinkedList Node类
Node是LinkedList的私有内部类，是LinkedList的核心，是LinkedList中用来存储节点的类，E 符号为泛型，属性item为当前的元素，next为指向当前节点下一个节点，prev为指向当前节点上一个节点，是一种双集合结构。
```
// Node 内部类 
private static class Node<E> {
    /** 当前存储的元素 */
    E item;
    /** 指向当前节点下一个节点 */
    Node<E> next;
    /** 指向当前节点上一个节点 */
    Node<E> prev;
    
    // 传入上一个节点，当前元素，下一个节点进行初始化。
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```
## LinkedList构造方法
```
// 构造一个空集合
public LinkedList() {
}

// 构造一个包含指定集合元素的集合，其顺序由集合的迭代器返回。
public LinkedList(Collection<? extends E> c) {
    // 调用无参构造函数进行初始化
    this();
    // 将集合c添加进集合
    addAll(c);
}
```
## LinkedList插入操作
### LinkedList插入尾部
将指定的元素添加到集合的末尾，该方法和addLast方法的作用一样，主要是通过linkLast 方法来实现插入到末尾，步骤如图所示。
```
// 将指定的元素添加到集合的末尾，此方法与addLast方法的作用一样
public boolean add(E e) {
    // 调用linkLast方法插入元素
    linkLast(e);
    return true;
}

// 将指定的元素添加到集合的末尾，此方法与add方法的作用一样 
public void addLast(E e) {
    linkLast(e);
}

// 将该元素添加到集合的末尾
void linkLast(E e) {
    // 获取旧尾节点
    final Node<E> l = last;
    // 构建一个新节点，该新节点的上一个节点指向旧尾节点，下一个节点为 null
    final Node<E> newNode = new Node<>(l, e, null);
    // 将新节点更新到尾节点
    last = newNode;
    
    // 如果旧尾节点为空，则该新节点既是首节点也是尾节点
    if (l == null)
        first = newNode;
    else
        // 旧尾节点不为空的话，将旧尾节点的下一个节点指向新尾节点
        l.next = newNode;
    // 集合长度 +1
    size++;
    // 修改次数 +1，适用于fail-fast迭代器
    modCount++;
}
```

![](/images/collection/LinkedList插入尾部.png)

### LinkedList插入首部
将该元素添加到集合的头部，主要通过调用linkFirst方法来实现，步骤如图所示。
```
// 将该元素添加到集合的头部
public void addFirst(E e) {
    linkFirst(e);
}

// 将该元素添加到集合的头部
private void linkFirst(E e) {
    // 获取集合旧首节点
    final Node<E> f = first;
    // 构建一个新节点，该新节点的下一个节点是旧首节点，上一个节点为 null
    final Node<E> newNode = new Node<>(null, e, f);
    // 将新节点更新到首节点
    first = newNode;
    
    // 如果旧首节点为空，则该新节点既是首节点也是尾节点
    if (f == null)
        last = newNode;
    else
        // 将旧首节点的上一个节点指向新首节点
        f.prev = newNode;
    // 集合长度 +1
    size++;
    // 修改次数 +1
    modCount++;
}
```

![](/images/collection/LinkedList插入首部.png)

### LinkedList插入指定位置
将指定的元素插入集合中的指定位置。将当前在该位置的元素（如果有的话）和任何后续的元素向右移位。在集合中间插入元素的平均时间复杂度为O(1)，该方式主要通过node(int index) 方法找到对应位置的节点，再通过linkBefore(E e, Node succ)方法进行插入，在集合中间插入的步骤如图所示。
```
// 将指定的元素插入集合中的指定位置
public void add(int index, E element) {
    // 检查index是否越界
    checkPositionIndex(index);

    // 如果index == size，则调用linkLast方法将该元素插入到最后一个
    if (index == size)
        linkLast(element);
    else
        // 将该节点插入到原来index位置的节点之前
        linkBefore(element, node(index));
}

// 检查下标是否越界
private void checkPositionIndex(int index) {
    // isPositionIndex返回false则抛出IndexOutOfBoundsException
    if (!isPositionIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

// 检查下标是否为迭代器或加法操作的有效位置的索引
private boolean isPositionIndex(int index) {
    // 下标小于 0 或 大于 size 返回 false
    return index >= 0 && index <= size;
}

// 在非null节点succ之前插入元素e，succ节点为下标index所在的节点，将包括该节点和该节点后面的节点往后移
void linkBefore(E e, Node<E> succ) {
    // 获取 succ 节点的上一个节点
    final Node<E> pred = succ.prev;
    // 构造新节点，该新节点的上一个节点为 pred，下一个节点为 succ
    final Node<E> newNode = new Node<>(pred, e, succ);
    // 将 succ 的前一个节点指向新节点
    succ.prev = newNode;
    
    // 如果 pred 节点为空，则插入的新节点指向首节点
    if (pred == null)
        first = newNode;
    else
        // pred 节点不为空，则 pred 节点的下一个节点指向新节点
        pred.next = newNode;
    // 集合长度 +1
    size++;
    // 修改次数 +1
    modCount++;
}
```

![](/images/collection/LinkedList插入指定位置.png)

## LinkedList删除操作
### LinkedList删除首部-remove()
删除集合的第一个节点，并返回该元素，与removeFirst方法的作用一样，主要通过 unlinkFirst方法实现删除头节点，并返回头节点的值，删除节点时将对应的节点值和节点的指向都置为了null，方便GC回收。删除步骤如图所示。
```
// 删除集合的第一个节点，并返回该元素
public E remove() {
    return removeFirst();
}

// 删除集合的第一个节点，并返回该元素
public E removeFirst() {
    // 获取首节点
    final Node<E> f = first;
    // 没有首节点抛出 NoSuchElementException
    if (f == null)
        throw new NoSuchElementException();
    // 删除首节点
    return unlinkFirst(f);
}

// 删除不为 null 的首节点
private E unlinkFirst(Node<E> f) {
    // 获取旧首节点的值
    final E element = f.item;
    // 获取旧首节点的下一个节点 next，next 为新首节点
    final Node<E> next = f.next;
    // 将旧首节点的值和下一个节点的指向赋为 null，帮助 GC
    f.item = null;
    f.next = null; 
    // 用新首节点 next 更新首节点 first
    first = next;
    // 如果 next 节点为空，则代表原集合只有一个节点，将尾节点也指向 null
    if (next == null)
        last = null;
    else
        // next 不为空的话，将该新首节点的上一个节点指向 null
        next.prev = null;
    // 集合长度 -1
    size--;
    // 修改次数 +1
    modCount++;
    // 返回删除的首节点元素
    return element;
}
```

![](/images/collection/LinkedList删除首部.png)


### LinkedList删除尾部-removeLast()
删除集合的最后一个节点，并返回该元素，主要通过unlinkLast方法实现删除尾节点，并返回尾节点的值，删除步骤如图所示。
```
// 删除尾节点并返回该元素
public E removeLast() {
    // 获取尾节点
    final Node<E> l = last;
    // 尾节点为 null 抛出 NoSuchElementException
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}

// 删除不为 null 的尾节点
private E unlinkLast(Node<E> l) {
    // 获取旧尾节点的值
    final E element = l.item;
    // 获取旧尾节点的上一个节点 prev，prev 为新尾节点
    final Node<E> prev = l.prev;
    // 将旧尾节点的值和下一个节点的指向置为 null，帮助 GC
    l.item = null;
    l.prev = null;
    // 用新尾节点 prev 更新尾节点 last
    last = prev;
    // 如果新尾节点节点为空，则代表该集合只有一个节点，将首节点指向 null
    if (prev == null)
        first = null;
    else
        // 新尾节点不为空，将新尾节点的下一个节点指向 null
        prev.next = null;
    // 集合长度 -1
    size--;
    // 修改次数 +1
    modCount++;
    // 返回删除的尾节点的值
    return element;
}
```

![](/images/collection/LinkedList删除尾部.png)

### LinkedList删除指定位置-remove(int index)
删除集合中指定位置的元素。将所有后续元素向前移动，并返回从集合中删除的元素，先通过 node(int index) 方法获取指定位置的节点，再通过unlink(Node x) 方法删除该节点并返回该节点的值，步骤如图所示。
```
// 删除集合中指定位置的元素。将所有后续元素向前移动，并返回从集合中删除的元素
public E remove(int index) {
    // 检查是否越界
    checkElementIndex(index);
    // 删除指定下标所在的节点
    return unlink(node(index));
}

// 删除指定不为 null 的节点
E unlink(Node<E> x) {
    // 获取指定节点的值，用于最后返回
    final E element = x.item;
    // 获取该节点的下一个节点 next
    final Node<E> next = x.next;
    // 获取该节点的上一个节点 prev
    final Node<E> prev = x.prev;

    // 如果 prev 节点为 null，表示该节点为首节点，将 next 节点指向首节点
    if (prev == null) {
        first = next;
    } else {
        // prev 节点不为 null，则将 prev 的下一个节点指向 next
        prev.next = next;
        // 将该节点的上一个节点置为 null，帮助 GC
        x.prev = null;
    }

    // 如果 next 节点为 null，表示该节点为尾节点，将 prev 节点指向尾节点
    if (next == null) {
        last = prev;
    } else {
        // next 节点不为 null，将 next 的上一个节点指向 prev
        next.prev = prev;
        // 将该节点的下一个节点置为 null，帮助 GC
        x.next = null;
    }
    // 将该节点的值置为 null
    x.item = null;
    // 集合长度 -1
    size--;
    // 修改次数 +1
    modCount++;
    // 返回删除的元素的值
    return element;
}
```

![](/images/collection/LinkedList删除尾部.png)

## LinkedList查询操作
### LinkedList查询指定位置-get(int index)
返回集合中指定位置的元素，先检查下标是否越界，再通过node(index)方法取到对应下标的节点，该节点的item属性即为对应的值。
```
// 返回集合中指定位置的元素
public E get(int index) {
    // 检查下标越界
    checkElementIndex(index);
    // node(index) 返回节点
    return node(index).item;
}

// 返回集合中的第一个元素
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}

// 返回集合中的最后一个元素
public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}
```
node(int index)返回指定元素索引处的（非空）元素，代码如下：
```
// 返回指定元素索引处的（非空）元素
Node<E> node(int index) {
    
    // 判断 index 更接近 0 还是 size 来决定从哪边遍历
    if (index < (size >> 1)) {
        // 从首节点遍历
        // 获取首节点
        Node<E> x = first;
        // 从首节点往后遍历 index 次，获取到 index 下标所在的节点
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        // 从尾节点遍历
        // 获取尾节点
        Node<E> x = last;
        // 从首节点往前遍历 size-index-1 次，获取到 index 下标所在的节点
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```
## 总结
LinkedList 底层是基于链表的，查找节点的平均时间复杂度是 O(n)，首尾增加和删除节点的时间复杂度是 O(1)。
LinkedList 适合读少写多的情况，ArrayList 适合读多写少的情况。
LinkedList 作为队列使用时，可以通过 offer/poll/peek 来代替 add/remove/get 等方法， 这些方法在遇到空集合或队列容量满的情况不会抛出异常。
