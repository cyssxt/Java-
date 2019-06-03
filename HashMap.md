# HashMap源码解析
## 内部类 Node
Node是HashMap是内部元素的存储结构，包含当前节点的哈希值，Key，value和下一个节点next
<pre><code>
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;//hash值
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);//将key和value的异或操作
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&//只有key和value都相等 对象才相等
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
</pre></code>

## 构造函数 初始容量initialCapacity和转载因子loadFactor（何时扩容的系数）
<pre><code>
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)//如果初始容量小于0 则抛出异常
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    //static final int MAXIMUM_CAPACITY = 1 << 30; HashMap的最大容量                                       
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))//判断是否是正常float类型
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);//设定扩容阈值 阈值为2的n次方
}

static final int tableSizeFor(int cap) {
    int n = cap - 1;//防止n是2的幂次方 示例8-9=7
    n |= n >>> 1;//这样000001110->000001111
    n |= n >>> 2;//000001111->000001111
    n |= n >>> 4;//000001111->000001111
    n |= n >>> 8;//000001111->000001111
    n |= n >>> 16;//000001111->000001111 
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;n+1=2的4次方
}
</code></pre>
## 一个参数构造函数 
<pre><code>
//参数为初始容量
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);//使用默认的装载因子
}
//参数为Map
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;//设置默认的装载因子
    putMapEntries(m, false);//将map放入新的HashMap
}

/**
 * Implements Map.putAll and Map constructor
 *
 * @param m the map
 * @param evict false when initially constructing this map, else
 * true (relayed to method afterNodeInsertion).
 */
 //transient Node<K,V>[] table;table就是Node的示例 也是一个HashMap最底层的存储结构
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        if (table == null) { // 如果当前table不存在
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                     (int)ft : MAXIMUM_CAPACITY);
            if (t > threshold)//如果容量大于阈值 ，则获取根据容量获取当前的阈值
                threshold = tableSizeFor(t);
        }
        else if (s > threshold)//如果map的长度 大于当前阈值 
            resize();//重新计算阈值
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {//循环设置值
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}

</code></pre>

## 没有参数的构造函数
<pre><code>

/**
 * Constructs an empty <tt>HashMap</tt> with the default initial capacity
 * (16) and the default load factor (0.75).
 */
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; //默认的转载因子
}
</code></pre>   

## put方法
<pre><code>
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)//如果当前table为null或者长度为0 则扩展容量
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null) //通过hash判断当前位置有没有元素 如果没有则创建 
        tab[i] = newNode(hash, key, value, null); 则创建一个新节点
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k)))) //如果不为null 但是key值相等 表示是同一个节点
            e = p;
        else if (p instanceof TreeNode)//如果p是treeNode 
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);//则将node放入tree中
        else {
            for (int binCount = 0; ; ++binCount) {//将节点放到最后一个节点的下一个节点
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st 判断树化阈值 ，如果同一个节点下 子节点过多 则将table转为树
                        treeifyBin(tab, hash);//将tab转为树
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key 如果已经存在
            V oldValue = e.value;//则替换旧值
            if (!onlyIfAbsent || oldValue == null) //onlyIfAbsent表示如果存在就不覆盖
                e.value = value;
            afterNodeAccess(e);
            return oldValue; //返回旧的值 
        }
    }
    ++modCount;//版本—+1
    if (++size > threshold)//长度大于阈值 则重新计算长度
        resize();
    afterNodeInsertion(evict);//回调函数
    return null;
}

final void treeifyBin(Node<K,V>[] tab, int hash) {//将tab树化
    int n, index; Node<K,V> e;
    static final int MIN_TREEIFY_CAPACITY = 64;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {//转为树
        TreeNode<K,V> hd = null, tl = null;
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);//用TreeNode替换Node
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
//将tab转为转为红黑树
final void treeify(Node<K,V>[] tab) {
    TreeNode<K,V> root = null;
    for (TreeNode<K,V> x = this, next; x != null; x = next) {
        next = (TreeNode<K,V>)x.next;
        x.left = x.right = null;
        if (root == null) {
            x.parent = null;
            x.red = false;
            root = x;
        }
        else {
            K k = x.key;
            int h = x.hash;
            Class<?> kc = null;
            for (TreeNode<K,V> p = root;;) {
                int dir, ph;
                K pk = p.key;
                if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((kc == null &&
                          (kc = comparableClassFor(k)) == null) ||
                         (dir = compareComparables(kc, k, pk)) == 0)
                    dir = tieBreakOrder(k, pk);

                TreeNode<K,V> xp = p;
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    x.parent = xp;
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    root = balanceInsertion(root, x);
                    break;
                }
            }
        }
    }
    moveRootToFront(tab, root);
}
</code></pre>

## 获取元素
<pre><code>
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {//在数组中查找
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);//在树中查找
            do {//否则 在链表中查找
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
</code></pre>
## contains 就是判断能不能获取到这个key
<pre><code>
public boolean containsKey(Object key) {
    return getNode(hash(key), key) != null;
}
</code></pre>

## resize重新计算长度
<pre><code>
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;//原表容量
    int oldThr = threshold;//阈值
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {//如果大于最大容量
            threshold = Integer.MAX_VALUE;//则取int最大值
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)//左移一位 符合条件
            newThr = oldThr << 1; // double threshold//新的容量和阈值都是扩大两倍
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;//如果旧容量为0 并且阈值大于0 则将阈值作为新的容量
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;//如果旧的容量和阈值都为0 则取默认的阈值和装载因子
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {//如果新的阈值为0 
        float ft = (float)newCap * loadFactor;//则通过容量和装载因子计算阈值
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {//如果旧的表格不为null ,则 重新分布表格节点
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
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
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
</code></pre>

## 删除节点
<pre><code>
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}
final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&//直接通过哈希值查找 
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {//如果有子节点
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);//在树上找节点
            else {
                do {//在链表上找节点
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
                             //如果找到 ，则直接删除
            if (node instanceof TreeNode)//如果是树节点  则通过树删除
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;//版本号+1
            --size;//长度-1
            afterNodeRemoval(node);//回调函数
            return node;
        }
    }
    return null;

</code></pre>

## 清除
<pre><code>
public void clear() {
    Node<K,V>[] tab;
    modCount++;//版本号+1
    if ((tab = table) != null && size > 0) {//表格存在 并且长度大于0
        size = 0;
        for (int i = 0; i < tab.length; ++i)//每项都改为null
            tab[i] = null;
    }
}
</code></pre>

## keySet
<pre><code>
public Set<K> keySet() {
    Set<K> ks = keySet;//先判断有没有keySet 没有就创建keySet
    if (ks == null) {
        ks = new KeySet();
        keySet = ks;
    }
    return ks;
}
public final Iterator<K> iterator()     { return new KeyIterator(); }
final class KeyIterator extends HashIterator
    implements Iterator<K> {
    public final K next() { return nextNode().key; }
}
final Node<K,V> nextNode() {
    Node<K,V>[] t;
    Node<K,V> e = next;
    if (modCount != expectedModCount)//判断版本号 非线程安全
        throw new ConcurrentModificationException();
    if (e == null)
        throw new NoSuchElementException();
    if ((next = (current = e).next) == null && (t = table) != null) {//遍历节点
        do {} while (index < t.length && (next = t[index++]) == null);
    }
    return e;
}
</code></pre>   