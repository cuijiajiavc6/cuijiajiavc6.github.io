```java 
package java.util;

import java.util.function.Consumer;
import java.util.function.BiConsumer;
import java.util.function.BiFunction;
import java.io.IOException;

/**
 * Map接口的哈希表和链表实现，具有可预测的迭代顺序。此实现与HashMap的不同之处在于它维护
 * 了一个贯穿其所有条目的双向链表。此链接列表定义迭代排序，通常是键插入映射的顺序（插
 * 入顺序）。请注意，如果将键重新插入地图，则不会影响插入顺序。 （如果m.containsKey（k）
 * 在调用之前立即返回true，则调用m.put（k，v）时，将密钥k重新插入到映射m中。）
 *
 *   这种实现使客户免于{@link HashMap}（以及{@link Hashtable}）提供的未指定的，通常是混乱的排序，
 * 而不会导致与{@link TreeMap}相关的成本增加。无论原始地图的实现如何，它都可用于生成与原始地图具有相同顺
 * 序的地图副本：
 *
 *  void foo（Map m）{Map copy = new LinkedHashMap（m）; ...}
 *
 *    如果模块在输入上获取地图，复制它，然后返回其顺序由副本确定的结果，则此技术特别有用。
 *  （客户通常会欣赏按照提交的顺序返回的内容。）
 *
 *   提供了一个特殊的{@link #LinkedHashMap（int，float，boolean）构造函数}来创建链接的哈希
 * 映射，其迭代顺序是其条目最后一次访问的顺序，从最近访问到最近访问（访问） - 订单）。这种地图非
 * 常适合构建LRU缓存。调用{@code put}，{@ code putIfAbsent}，{@ code get}，
 * {@ code getOrDefault}，{@ code compute}，{@ code computeIfAbsent}，{@ code computeIfPresent}
 * 或{@code merge}方法导致访问相应的条目（假设它在调用完成后存在）。
 *  {@code replace}方法仅在替换值时才会访问该条目。 {@code putAll}方法为指定映射中的每个映射生成一
 *  个条目访问，按照指定映射的条目集迭代器提供键 -
 *  值映射的顺序。没有其他方法可以生成条目访问。特别是，对集合视图的操作不会影响后备映射的
 *  迭代顺序。
 *
 *   可以重写{@link #removeEldestEntry（Map.Entry）}方法，以强制在将新映射添加到地图时自动删除过
 * 时映射的策略。
 *
 *   此类提供所有可选的Map操作，并允许null元素。与HashMap一样，它为基本操作（添加，包含和
 * 删除）提供了恒定时间性能，假设哈希函数在桶之间正确地分散元素。由于维护链表的额外费用，
 * 性能可能略低于HashMap的性能，但有一个例外：对LinkedHashMap的集合视图进行迭代需要与映射大
 * 小成比例的时间，无论其容量如何。对HashMap的迭代可能更昂贵，需要与其容量成比例的时间。
 *
 *   链接的哈希映射有两个影响其性能的参数：初始容量和负载因子。它们的定义与HashMap完全相
 * 同。但请注意，为此类选择过高的初始容量值的惩罚不如HashMap严重，因为此类的迭代时间不受
 * 容量影响。
 *
 *    请注意，此实现不同步。如果多个线程同时访问链接的哈希映射，并且至少有一个线程在结构
 * 上修改了映射，则必须在外部进行同步。这通常通过在自然封装地图的某个对象上进行同步来实现。
 *
 *  如果不存在此类对象，则应使用{@link Collections＃synchronizedMap Collections.synchronizedMap}
 * 方法“包装”映射。这最好在创建时完成，以防止对映射的意外不同步访问：
 * Map m = Collections.synchronizedMap（new LinkedHashMap（...））;
 *
 *  结构修改是添加或删除一个或多个映射的任何操作，或者在访问顺序链接的哈希映射的情况下，
 * 影响迭代顺序。在插入有序链接散列映射中，仅更改与已包含在映射中的键相关联的值不是结构
 * 修改。在访问有序链接哈希映射中，仅使用get查询映射是结构修改。 ）
 *
 *
 *   所有此类的集合视图方法返回的集合的迭代器方法返回的迭代器都是快速失败的：如果在创建
 * 迭代器之后的任何时候对映射进行结构修改，除非通过迭代器自己的remove方法，迭代器将抛出
 * {@link ConcurrentModificationException}。因此，面对并发修改，迭代器会快速而干净地失
 * 败，而不是冒着任意的非确定性行为的风险。
 * 
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @implNote
 * The spliterators returned by the spliterator method of the collections
 * returned by all of this class's collection view methods are created from
 * the iterators of the corresponding collections.
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 *
 * @author  Josh Bloch
 * @see     Object#hashCode()
 * @see     Collection
 * @see     Map
 * @see     HashMap
 * @see     TreeMap
 * @see     Hashtable
 * @since   1.4
 */
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
{

	/**
	 * 实施说明。 此类的先前版本的内部结构略有不同。
	 * 因为超类HashMap现在为其某些节点使用树，所以现在将
	 * LinkedHashMap.Entry类视为中间节点类，也可以将其转换为树形式。
	 * 此类的名称LinkedHashMap.Entry在其当前上下文中以多种方式混淆，
	 * 但无法更改。 否则，即使它未被导出到此包之外，已知一些现有的
	 * 源代码依赖于对removeEldestEntry的调用中的符号解析角案例规则，
	 * 该删除由于模糊使用而抑制了编译错误。 因此，我们保留名称以保
	 * 留未修改的可编译性。 节点类的更改还需要使用两个字段（头部，尾部）
	 * 而不是指向标头节点的指针来维护列表之前/之后的双向链接。 此类以前
	 * 还在访问，插入和删除时使用了不同类型的回调方法。
	 * */

    /**
     * 正常LinkedHashMap条目的HashMap.Node子类。
     */
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }

    private static final long serialVersionUID = 3801124242820219131L;

    /**
     * 双向链表的头（最长）。
     */
    transient LinkedHashMap.Entry<K,V> head;

    /**
     * 双向链表的尾部（最年轻）。
     */
    transient LinkedHashMap.Entry<K,V> tail;

    /**
     * 此链接哈希映射的迭代排序方法：对于访问顺序为true，对于插入顺序为false。
     *
     * @serial
     */
    final boolean accessOrder;

    // internal utilities

    // link at the end of list
    private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }

    // apply src's links to dst
    private void transferLinks(LinkedHashMap.Entry<K,V> src,
                               LinkedHashMap.Entry<K,V> dst) {
        LinkedHashMap.Entry<K,V> b = dst.before = src.before;
        LinkedHashMap.Entry<K,V> a = dst.after = src.after;
        if (b == null)
            head = dst;
        else
            b.after = dst;
        if (a == null)
            tail = dst;
        else
            a.before = dst;
    }

    // overrides of HashMap hook methods

    void reinitialize() {
        super.reinitialize();
        head = tail = null;
    }

    Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMap.Entry<K,V> p =
            new LinkedHashMap.Entry<K,V>(hash, key, value, e);
        linkNodeLast(p);
        return p;
    }

    Node<K,V> replacementNode(Node<K,V> p, Node<K,V> next) {
        LinkedHashMap.Entry<K,V> q = (LinkedHashMap.Entry<K,V>)p;
        LinkedHashMap.Entry<K,V> t =
            new LinkedHashMap.Entry<K,V>(q.hash, q.key, q.value, next);
        transferLinks(q, t);
        return t;
    }

    TreeNode<K,V> newTreeNode(int hash, K key, V value, Node<K,V> next) {
        TreeNode<K,V> p = new TreeNode<K,V>(hash, key, value, next);
        linkNodeLast(p);
        return p;
    }

    TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
        LinkedHashMap.Entry<K,V> q = (LinkedHashMap.Entry<K,V>)p;
        TreeNode<K,V> t = new TreeNode<K,V>(q.hash, q.key, q.value, next);
        transferLinks(q, t);
        return t;
    }

    void afterNodeRemoval(Node<K,V> e) { // unlink
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.before = p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a == null)
            tail = b;
        else
            a.before = b;
    }

    void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }

    void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }

    void internalWriteEntries(java.io.ObjectOutputStream s) throws IOException {
        for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after) {
            s.writeObject(e.key);
            s.writeObject(e.value);
        }
    }

    /**
     * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
     * with the specified initial capacity and load factor.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public LinkedHashMap(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;
    }

    /**
     * 构造一个具有指定初始容量和默认加载因子（0.75）的空插入排序LinkedHashMap实例。
     *
     * @param  initialCapacity the initial capacity
     * @throws IllegalArgumentException if the initial capacity is negative
     */
    public LinkedHashMap(int initialCapacity) {
        super(initialCapacity);
        accessOrder = false;
    }

    /**
     * 使用默认初始容量（16）和加载因子（0.75）构造一个空的插入顺序LinkedHashMap实例。
     */
    public LinkedHashMap() {
        super();
        accessOrder = false;
    }

    /**
     * 构造一个插入有序的LinkedHashMap实例，其具有与指定映射相同的映射。 
     * LinkedHashMap实例使用默认加载因子（0.75）和足以保存指定映射中的映射的初始容量创建。
     *
     * @param  m the map whose mappings are to be placed in this map
     * @throws NullPointerException if the specified map is null
     */
    public LinkedHashMap(Map<? extends K, ? extends V> m) {
        super();
        accessOrder = false;
        putMapEntries(m, false);
    }

    /**
     * 使用指定的初始容量，加载因子和排序模式构造一个空的LinkedHashMap实例。
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @param  accessOrder     the ordering mode - <tt>true</tt> for
     *         access-order, <tt>false</tt> for insertion-order
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }


    /**
     * 如果此映射将一个或多个键映射到指定值，则返回true。
     *
     * @param value value whose presence in this map is to be tested
     * @return <tt>true</tt> if this map maps one or more keys to the
     *         specified value
     */
    public boolean containsValue(Object value) {
        for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after) {
            V v = e.value;
            if (v == value || (value != null && value.equals(v)))
                return true;
        }
        return false;
    }

	/**
	 * 返回指定键映射到的值，如果此映射不包含键的映射，则返回{@code null}。
	 *  更正式地说，如果此映射包含从密钥{@code k}到值{@code v}的映射，使得
	 *  {@code（key == null？k == null：key.equals（k））}， 然后这个方法返
	 *  回{@code v}; 否则返回{@code null}。 （最多可以有一个这样的映射。）
	 *  {@code null}的返回值不一定表示映射不包含密钥的映射; 地图也可能明确地
	 *  将密钥映射到{@code null}。 {@link #containsKey containsKey}操作可用
	 *  于区分这两种情况。
	 * */
    public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }

    /**
     * {@inheritDoc}
     */
    public V getOrDefault(Object key, V defaultValue) {
       Node<K,V> e;
       if ((e = getNode(hash(key), key)) == null)
           return defaultValue;
       if (accessOrder)
           afterNodeAccess(e);
       return e.value;
   }

    /**
     * {@inheritDoc}
     */
    public void clear() {
        super.clear();
        head = tail = null;
    }

    /**
     * 如果此映射应删除其最旧条目，则返回true。在将新条目插入地图后，put和putAll
     *将调 用此方法。它为实现者提供了在每次添加新条目时删除最旧条目的机会。如果
     * 映射表示 高速缓存，则此选项非常有用：它允许映射通过删除过时条目来减少内存
     * 消耗。 示例使 用：此覆盖将允许地图增长到100个条目，然后在每次添加新条目时
     * 删除最旧的条目， 保持100个条目的稳定状态。
     * <pre>
     *     private static final int MAX_ENTRIES = 100;
     *
     *     protected boolean removeEldestEntry(Map.Entry eldest) {
     *        return size() &gt; MAX_ENTRIES;
     *     }
     * </pre>
     *
     *   此方法通常不以任何方式修改地图，而是允许地图根据其返回值进行修改。此方法
     * 允许直接修改映射，但如果它这样做，则必须返回false（表示映射不应尝试任何进一
     * 步的修改）。在此方法中修改地图后返回true的效果未指定。此实现仅返回false（因此
     * 此映射的行为类似于普通映射 - 永远不会删除最旧的元素）。
     *
     * @param    eldest The least recently inserted entry in the map, or if
     *           this is an access-ordered map, the least recently accessed
     *           entry.  This is the entry that will be removed it this
     *           method returns <tt>true</tt>.  If the map was empty prior
     *           to the <tt>put</tt> or <tt>putAll</tt> invocation resulting
     *           in this invocation, this will be the entry that was just
     *           inserted; in other words, if the map contains a single
     *           entry, the eldest entry is also the newest.
     * @return   <tt>true</tt> if the eldest entry should be removed
     *           from the map; <tt>false</tt> if it should be retained.
     */
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
    }

	/**
	 * 返回此映射中包含的键的{@link Set}视图。 该集由地图支持，因此对地图的更改将反映在集
	 * 中，反之亦然。 如果在对集合进行迭代时修改了映射（除了通过迭代器自己的remove操作），
	 * 迭代的结果是未定义的。 该集支持元素删除，它通过Iterator.remove，Set.remove，
	 * removeAll，retainAll和clear操作从地图中删除相应的映射。 它不支持add或addAll操作。
	 *  它的{@link Spliterator}通常提供更快的顺序性能，但并行性能比{@code HashMap}差得多。
	 * 
     *
     * @return a set view of the keys contained in this map
     */
    public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new LinkedKeySet();
            keySet = ks;
        }
        return ks;
    }

    final class LinkedKeySet extends AbstractSet<K> {
        public final int size()                 { return size; }
        public final void clear()               { LinkedHashMap.this.clear(); }
        public final Iterator<K> iterator() {
            return new LinkedKeyIterator();
        }
        public final boolean contains(Object o) { return containsKey(o); }
        public final boolean remove(Object key) {
            return removeNode(hash(key), key, null, false, true) != null;
        }
        public final Spliterator<K> spliterator()  {
            return Spliterators.spliterator(this, Spliterator.SIZED |
                                            Spliterator.ORDERED |
                                            Spliterator.DISTINCT);
        }
        public final void forEach(Consumer<? super K> action) {
            if (action == null)
                throw new NullPointerException();
            int mc = modCount;
            for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after)
                action.accept(e.key);
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }

	/**
	 * 返回此映射中包含的值的{@link Collection}视图。该集合由地图支持，因此对地图
	 * 的更改将反映在集合中，反之亦然。如果在对集合进行迭代时修改了映射（除了通过
	 * 迭代器自己的remove操作），迭代的结果是未定义的。该集合支持元素删除，它通过
	 * Iterator.remove，Collection.remove，removeAll，retainAll和clear操作从地图中
	 * 删除相应的映射。它不支持add或addAll操作。它的{@link Spliterator}通常提供更
	 * 快的顺序性能，但并行性能比{@code HashMap}差得多。
     *
     * @return a view of the values contained in this map
     */
    public Collection<V> values() {
        Collection<V> vs = values;
        if (vs == null) {
            vs = new LinkedValues();
            values = vs;
        }
        return vs;
    }

    final class LinkedValues extends AbstractCollection<V> {
        public final int size()                 { return size; }
        public final void clear()               { LinkedHashMap.this.clear(); }
        public final Iterator<V> iterator() {
            return new LinkedValueIterator();
        }
        public final boolean contains(Object o) { return containsValue(o); }
        public final Spliterator<V> spliterator() {
            return Spliterators.spliterator(this, Spliterator.SIZED |
                                            Spliterator.ORDERED);
        }
        public final void forEach(Consumer<? super V> action) {
            if (action == null)
                throw new NullPointerException();
            int mc = modCount;
            for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after)
                action.accept(e.value);
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }

    /**
     * 返回此映射中包含的映射的{@link Set}视图。该集由地图支持，因此对地图的更改
	 * 将反映在集中，反之亦然。如果在对集合进行迭代时修改了映射（除非通过迭代器自
	 * 己的remove操作，或者通过迭代器返回的映射条目上的setValue操作），迭代的结果
	 * 是未定义的。该集支持元素删除，它通过Iterator.remove，Set.remove，removeAll
	 * ，retainAll和clear操作从地图中删除相应的映射。它不支持add或addAll操作。它
	 * 的{@link Spliterator}通常提供更快的顺序性能，但并行性能比{@code HashMap}差
	 * 得多。
     *
     * @return a set view of the mappings contained in this map
     */
    public Set<Map.Entry<K,V>> entrySet() {
        Set<Map.Entry<K,V>> es;
        return (es = entrySet) == null ? (entrySet = new LinkedEntrySet()) : es;
    }

    final class LinkedEntrySet extends AbstractSet<Map.Entry<K,V>> {
        public final int size()                 { return size; }
        public final void clear()               { LinkedHashMap.this.clear(); }
        public final Iterator<Map.Entry<K,V>> iterator() {
            return new LinkedEntryIterator();
        }
        public final boolean contains(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry<?,?>) o;
            Object key = e.getKey();
            Node<K,V> candidate = getNode(hash(key), key);
            return candidate != null && candidate.equals(e);
        }
        public final boolean remove(Object o) {
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>) o;
                Object key = e.getKey();
                Object value = e.getValue();
                return removeNode(hash(key), key, value, true, true) != null;
            }
            return false;
        }
        public final Spliterator<Map.Entry<K,V>> spliterator() {
            return Spliterators.spliterator(this, Spliterator.SIZED |
                                            Spliterator.ORDERED |
                                            Spliterator.DISTINCT);
        }
        public final void forEach(Consumer<? super Map.Entry<K,V>> action) {
            if (action == null)
                throw new NullPointerException();
            int mc = modCount;
            for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after)
                action.accept(e);
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }

    // Map overrides

    public void forEach(BiConsumer<? super K, ? super V> action) {
        if (action == null)
            throw new NullPointerException();
        int mc = modCount;
        for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after)
            action.accept(e.key, e.value);
        if (modCount != mc)
            throw new ConcurrentModificationException();
    }

    public void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
        if (function == null)
            throw new NullPointerException();
        int mc = modCount;
        for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after)
            e.value = function.apply(e.key, e.value);
        if (modCount != mc)
            throw new ConcurrentModificationException();
    }

    // Iterators

    abstract class LinkedHashIterator {
        LinkedHashMap.Entry<K,V> next;
        LinkedHashMap.Entry<K,V> current;
        int expectedModCount;

        LinkedHashIterator() {
            next = head;
            expectedModCount = modCount;
            current = null;
        }

        public final boolean hasNext() {
            return next != null;
        }

        final LinkedHashMap.Entry<K,V> nextNode() {
            LinkedHashMap.Entry<K,V> e = next;
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            if (e == null)
                throw new NoSuchElementException();
            current = e;
            next = e.after;
            return e;
        }

        public final void remove() {
            Node<K,V> p = current;
            if (p == null)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            current = null;
            K key = p.key;
            removeNode(hash(key), key, null, false, false);
            expectedModCount = modCount;
        }
    }

    final class LinkedKeyIterator extends LinkedHashIterator
        implements Iterator<K> {
        public final K next() { return nextNode().getKey(); }
    }

    final class LinkedValueIterator extends LinkedHashIterator
        implements Iterator<V> {
        public final V next() { return nextNode().value; }
    }

    final class LinkedEntryIterator extends LinkedHashIterator
        implements Iterator<Map.Entry<K,V>> {
        public final Map.Entry<K,V> next() { return nextNode(); }
    }


}

```