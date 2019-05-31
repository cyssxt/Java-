# 前言
<p>ArrayList是List的一个实现，主要用于动态列表。它是一个有序的、非线程安全的可变有序列表</p>

# 构造函数

## 初始长度的构造函数
<pre><code>
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) { //如果长度大于0 则创建一个长度为initialCapacity的object数组
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
        //private static final Object[] EMPTY_ELEMENTDATA = {}; 此处是定义的默认空数组
    } else {
        //如果长度小于 则异常
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
</code></pre>
## 无参构造函数
<pre><code>
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    //private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {}; 也是直接创建一个空数
}
</code></pre>

## 容器构造函数
<pre><code>
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();//将传入的容器转化为Object数组
    if ((size = elementData.length) != 0) {//如果数组长度不等于0
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            //并且数组类型不是对象数组 则拷贝数组对象为Object[]类型 因为elementData为Object[]数组 非范型数组
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
</code></pre>

## 注意 
   List中的modCount类似版本号，防止操作的对象版本号不一致 而引发的线程安全问题
   
## add操作

<pre><code>
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!! 
    //使用当前数组长度+1计算长度
    elementData[size++] = e;//设置值
    return true; //只要没有异常则为true
}

private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {//如果数组为空 
        return Math.max(DEFAULT_CAPACITY, minCapacity); 
        //则将最小容量和和默认的容量比较（DEFAULT_CAPACITY=10） 返回最小值
        //此处可以发现当我们调用无参的构造函数时，首次添加时候，会将列表长度增至10
    }
    return minCapacity;
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;//自增位移量 次数为当前列表中的元素个数

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)//如果最小容量大于数组的长度 则扩展容量
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1); //计算新的容量 右移动一位 相当于扩展1/2
    if (newCapacity - minCapacity < 0) //如果扩展1/2的容量小于最小容量 则取最小容量
        newCapacity = minCapacity;
    //private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    if (newCapacity - MAX_ARRAY_SIZE > 0) //如果新容量大于最大长度 
        newCapacity = hugeCapacity(minCapacity);//判断溢出 ，如果大于MAX_ARRAY_SIZE 则设置为Integer.MAX_VALUE
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);//拷贝拷贝新数组
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow 判断溢出
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
</code></pre>
## 获取list长度
<pre><code>
public int size() {
    checkForComodification();//校验modCount
    return this.size;
}

private void checkForComodification() {
    if (ArrayList.this.modCount != this.modCount)//如果不等抛出异常 次数为非线程安全的原因
        throw new ConcurrentModificationException();
}
</code></pre>
## 获取list中元素
<pre><code>
public E get(int index) {
    rangeCheck(index);//判断下标是否超出列表长度 超出抛出异常
    checkForComodification();//校验modCount
    return ArrayList.this.elementData(offset + index);//然后数组中的长度
}

private void rangeCheck(int index) {
    if (index < 0 || index >= this.size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

private String outOfBoundsMsg(int index) {
    return "Index: "+index+", Size: "+this.size;//异常信息为下标和列表长度
}

</code></pre>

## 更新list中元素
<pre><code>
public E set(int index, E element) {
    rangeCheck(index);//下标校验 如果超出 则抛出异常

    E oldValue = elementData(index);//获取旧的元素
    elementData[index] = element;//更新元素
    return oldValue;//返回旧元素
}
private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
</code></pre>

## 根据下标删除元素
<pre><code>
public E remove(int index) {
    rangeCheck(index);//校验元素

    modCount++;//版本号+1
    E oldValue = elementData(index);//获取旧的元素

    int numMoved = size - index - 1;//获取需要移动数量
    if (numMoved > 0)//然后拷贝
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work 缩小长度

    return oldValue; //返回之前的值
}
</code></pre>

## 根据对象删除
<pre><code>
public boolean remove(Object o) {
    if (o == null) { //如果为null 删除所有null值元素
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {//否则 一条条删除
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work 删除数组最后一项 并且更新数组长度 此时 数组的真正长度没变
}
</code></pre>
<font color='red'>ps:如果add的是int值 是无法通过对象删除的</font>
## 批量删除
<pre><code>
public boolean removeAll(Collection<?> c) {
    Objects.requireNonNull(c);//集合不能为null
    return batchRemove(c, false);
}

private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    int r = 0, w = 0;
    boolean modified = false;//修改完成标志
    try {
        for (; r < size; r++)//逐项比较
            if (c.contains(elementData[r]) == complement)//判断包含 双指针删除
                elementData[w++] = elementData[r];
    } finally {
        // Preserve behavioral compatibility with AbstractCollection,
        // even if c.contains() throws.
        if (r != size) {//修改长度
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            w += size - r;
        }
        if (w != size) {
            // clear to let GC do its work
            for (int i = w; i < size; i++)
                elementData[i] = null; //清楚gc
            modCount += size - w;//修改版本
            size = w;
            modified = true;
        }
    }
    return modified;//修改完成返回true
}
</code></pre>

## 清空
<pre><code>
public void clear() {
    modCount++;//清空版本

    // clear to let GC do its work
    for (int i = 0; i < size; i++)//逐项置为空 并且标记为null
        elementData[i] = null;

    size = 0;
}
</code></pre>

## 获取元素下标
<pre><code>
public int indexOf(Object o) {
    if (o == null) {//元素为null,则返回null值
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {//不为null 则判断equals
        for (int i = 0; i < size; i++) 
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;//找不到 返回-1
}
</code></pre>

## 获取下标
<pre><code>
public boolean contains(Object o) {
    return indexOf(o) >= 0; //判断元素下标是否为0
}
</code></pre>

## 列表相减
<pre><code>
public List<E> subList(int fromIndex, int toIndex) {
    subListRangeCheck(fromIndex, toIndex, size);//校验下标
    return new SubList(this, 0, fromIndex, toIndex);//返回subList 为内部类
}

static void subListRangeCheck(int fromIndex, int toIndex, int size) {
    if (fromIndex < 0)
        throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
    if (toIndex > size)
        throw new IndexOutOfBoundsException("toIndex = " + toIndex);
    if (fromIndex > toIndex)
        throw new IllegalArgumentException("fromIndex(" + fromIndex +
                                           ") > toIndex(" + toIndex + ")");
}
</code></pre>
## 序列化
<pre><code>
private void writeObject(java.io.ObjectOutputStream s);//序列化
private void readObject(java.io.ObjectInputStream s);// 反序列化
// 由于elementData是不序列化的 所以开启了
 </code></pre>
 
## iterator
<pre><code>
public Iterator<E> iterator() {
    return new Itr();//生成Iterator迭代对象
}

private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return 指针
    int lastRet = -1; // index of last element returned; -1 if no such //上一个元素
    int expectedModCount = modCount;//版本号

    Itr() {}

    public boolean hasNext() {
        return cursor != size;//判断指针是否和列表相等
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();//校验版本号 这就是在遍历的过程中 为何不要改变元素内容
        int i = cursor;//设置下标
        if (i >= size)//超出长度 抛出异常 终止遍历
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)//判断长度是否一致  主要避免多线程修改的问题
            throw new ConcurrentModificationException();
        cursor = i + 1;//指针后移
        return (E) elementData[lastRet = i];//返回当前的元素
    }

    public void remove() {//通过上次的指针 删除元素
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();//校验版本号

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public void forEachRemaining(Consumer<? super E> consumer) {
        Objects.requireNonNull(consumer);//校验消费者是否为null
        final int size = ArrayList.this.size;
        int i = cursor;
        if (i >= size) {
            return;
        }
        final Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length) {
            throw new ConcurrentModificationException();
        }
        while (i != size && modCount == expectedModCount) {
            consumer.accept((E) elementData[i++]);
        }
        // update once at end of iteration to reduce heap write traffic
        cursor = i;
        lastRet = i - 1;
        checkForComodification();
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
PS:forEachRemaining可以直接遍历元素 并且消费
</code></pre>

## 排序方法
<pre><code>
public void sort(Comparator<? super E> c) {
    final int expectedModCount = modCount;//记录版本好 防止排序过程中 被其他线程修改
    Arrays.sort((E[]) elementData, 0, size, c);
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
    modCount++;//新增版本号
}
</code></pre>

## forEach和iterator中的forEachRemaining类似
<pre><code>
public void forEach(Consumer<? super E> action) {
    Objects.requireNonNull(action);
    final int expectedModCount = modCount;
    @SuppressWarnings("unchecked")
    final E[] elementData = (E[]) this.elementData;
    final int size = this.size;
    for (int i=0; modCount == expectedModCount && i < size; i++) {
        action.accept(elementData[i]);
    }
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
</code></pre>

## 条件删除
<pre><code>
 public boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    // figure out which elements are to be removed
    // any exception thrown from the filter predicate at this stage
    // will leave the collection unmodified
    int removeCount = 0;
    final BitSet removeSet = new BitSet(size);
    final int expectedModCount = modCount;
    final int size = this.size;
    for (int i=0; modCount == expectedModCount && i < size; i++) {
        @SuppressWarnings("unchecked")
        final E element = (E) elementData[i];
        if (filter.test(element)) {//条件判断
            removeSet.set(i);
            removeCount++;
        }
    }
    if (modCount != expectedModCount) {//版本校验
        throw new ConcurrentModificationException();
    }

    // shift surviving elements left over the spaces left by removed elements
    final boolean anyToRemove = removeCount > 0;
    if (anyToRemove) {//如果有删除的 则修改长度 删除元素
        final int newSize = size - removeCount;
        for (int i=0, j=0; (i < size) && (j < newSize); i++, j++) {
            i = removeSet.nextClearBit(i);
            elementData[j] = elementData[i];
        }
        for (int k=newSize; k < size; k++) {
            elementData[k] = null;  // Let gc do its work
        }
        this.size = newSize;
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }

    return anyToRemove;//返回修改的长度
}
</code></pre>

## public void replaceAll(UnaryOperator<E> operator);批量替换

## sublist结构 结构ArrayList只是内部多了parent，偏移量