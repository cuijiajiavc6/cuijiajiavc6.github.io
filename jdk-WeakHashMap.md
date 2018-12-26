```java 

package java.util;

import java.lang.ref.WeakReference;
import java.lang.ref.ReferenceQueue;
import java.util.concurrent.ThreadLocalRandom;
import java.util.function.BiConsumer;
import java.util.function.BiFunction;
import java.util.function.Consumer;


/**
 * 基于哈希表的Map接口实现，带弱键。当WeakHashMap的密钥不再正常使用时，
 * 它将自动被删除。更确切地说，给定密钥的映射的存在不会阻止密钥被垃圾收
 * 集器丢弃，即，可以最终化，最终化，然后回收。当一个键被丢弃时，它的条
 * 目将被有效地从地图中删除，因此该类的行为与其他Map实现略有不同。支持空
 * 值和空值。该类具有与HashMap类相似的性能特征，并具有与初始容量和负载因
 * 子相同的效率参数。与大多数集合类一样，此类不同步。可以使用
 * {@link Collections＃synchronizedMap Collections.synchronizedMap}方法
 * 构造同步的WeakHashMap。此类主要用于使用等于方法的关键对象，使用==运算
 * 符测试对象标识。一旦这样的密钥被丢弃，它就永远不会被重新创建，因此以
 * 后不可能在WeakHashMap中查找该密钥，并且对其条目已被删除感到惊讶。这
 * 个类可以很好地处理其equals方法不基于对象标识的关键对象，例如String
 * 实例。但是，使用这种可重新调用的密钥对象，自动删除其密钥已被丢弃的
 * WeakHashMap条目可能会令人困惑。 WeakHashMap类的行为部分取决于垃圾
 * 收集器的操作，因此几个熟悉的（但不是必需的）Map不变量不适用于此类。
 * 由于垃圾收集器可能随时丢弃密钥，因此WeakHashMap的行为可能就像未知
 * 线程正在静默删除条目一样。特别是，即使您在WeakHashMap实例上进行同步
 * 并且不调用其mutator方法，size方法也可能会随着时间的推移返回较小的值，
 * 因为isEmpty方法返回false然后返回true，以便containsKey方法返回对于给定键，
 * true和更高版本为false，因为get方法返回给定键的值但后来返回null，put方法
 * 返回null，remove方法返回false，前面看来是在映射，以及对密钥集，值集合和
 * 条目集的连续检查，以连续产生较少数量的元素。 WeakHashMap中的每个关键对
 * 象都间接存储为弱引用的引用对象。因此，只有在垃圾收集器清除了对映射内部
 * 和外部的弱引用之后，才会自动删除密钥。实现说明：WeakHashMap中的值对象
 * 由普通的强引用保存。因此，应该注意确保值对象不直接或间接地强烈引用它们
 * 自己的密钥，因为这将防止密钥被丢弃。请注意，值对象可以通过WeakHashMap
 * 本身间接引用其键;也就是说，值对象可以强烈地引用一些其他关键对象，其关
 * 联的值对象又强烈地引用第一值对象的关键字。如果映射中的值不依赖于持有对
 * 它们的强引用的映射，则处理此问题的一种方法是在插入之前将值本身包装在
 * WeakReferences中，如：m.put（key，new WeakReference（value）），然后解
 * 开每一个。所有这个类的“集合视图方法”返回的集合的迭代器方法返回的迭代器
 * 是快速失败的：如果在创建迭代器之后的任何时候对映射进行结构修改，除非通
 * 过迭代器自己的删除方法，迭代器将抛出{@link ConcurrentModificationException}。
 * 因此，在并发修改的情况下，迭代器快速而干净地失败，而不是在未来的未确定时间
 * 冒任意，非确定性行为的风险。请注意，迭代器的快速失败行为无法得到保证，因
 * 为一般来说，在存在不同步的并发修改时，不可能做出任何硬性保证。失败快速迭
 * 代器会尽最大努力抛出ConcurrentModificationException。因此，编写依赖于此
 * 异常的程序以确保其正确性是错误的：迭代器的快速失败行为应该仅用于检测错误。
 * 此类是Java Collections Framework的成员。
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 *
 * @author      Doug Lea
 * @author      Josh Bloch
 * @author      Mark Reinhold
 * @since       1.2
 * @see         java.util.HashMap
 * @see         java.lang.ref.WeakReference
 */
public class WeakHashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V> {

    /**
     * 默认初始容量 - 必须是2的幂。
     */
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    /**
     * 如果具有参数的任一构造函数隐式指定较高值，则使用最大容量。 必须是两个<= 1 << 
     * 30的幂。     
     */
    private static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 在构造函数中未指定时使用的加载因子。
     */
    private static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * 该表根据需要调整大小。 长度必须始终是2的幂。
     */
    Entry<K,V>[] table;

    /**
     * 此弱哈希映射中包含的键 - 值映射的数量。
     */
    private int size;

    /**
     * 要调整大小的下一个大小值（capacity * load factor）。
     */
    private int threshold;

    /**
     * 哈希表的加载因子。
     */
    private final float loadFactor;

    /**
     * 已清除的WeakEntries的引用队列
     */
    private final ReferenceQueue<Object> queue = new ReferenceQueue<>();

    /**
     * 此WeakHashMap已被结构修改的次数。 
     * 结构修改是那些改变映射中映射数量或以其他方式修改其内部结构
     *（例如，重新散列）的修改。 此字段用于使映射的Collection-views上的迭代器快速失败。
     *
     * @see ConcurrentModificationException
     */
    int modCount;

    @SuppressWarnings("unchecked")
    private Entry<K,V>[] newTable(int n) {
        return (Entry<K,V>[]) new Entry<?,?>[n];
    }

    /**
     * 使用给定的初始容量和给定的加载因子构造一个新的空WeakHashMap。
     *
     * @param  initialCapacity The initial capacity of the <tt>WeakHashMap</tt>
     * @param  loadFactor      The load factor of the <tt>WeakHashMap</tt>
     * @throws IllegalArgumentException if the initial capacity is negative,
     *         or if the load factor is nonpositive.
     */
    public WeakHashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Initial Capacity: "+
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;

        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal Load factor: "+
                                               loadFactor);
        int capacity = 1;
        while (capacity < initialCapacity)
            capacity <<= 1;
        table = newTable(capacity);
        this.loadFactor = loadFactor;
        threshold = (int)(capacity * loadFactor);
    }

    /**
     * 使用给定的初始容量和默认加载因子（0.75）构造一个新的空WeakHashMap。
     *
     * @param  initialCapacity The initial capacity of the <tt>WeakHashMap</tt>
     * @throws IllegalArgumentException if the initial capacity is negative
     */
    public WeakHashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * 使用默认初始容量（16）和加载因子（0.75）构造一个新的空WeakHashMap。
     */
    public WeakHashMap() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
    }

    /**
     * 使用与指定映射相同的映射构造一个新的WeakHashMap。 使用默认加载因子
     *（0.75）创建WeakHashMap，初始容量足以保存指定映射中的映射。
     *
     * @param   m the map whose mappings are to be placed in this map
     * @throws  NullPointerException if the specified map is null
     * @since   1.3
     */
    public WeakHashMap(Map<? extends K, ? extends V> m) {
        this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                DEFAULT_INITIAL_CAPACITY),
             DEFAULT_LOAD_FACTOR);
        putAll(m);
    }

    // internal utilities

    /**
     * 表示表内的空键的值。
     */
    private static final Object NULL_KEY = new Object();

    /**
     * Use NULL_KEY for key if it is null.
     */
    private static Object maskNull(Object key) {
        return (key == null) ? NULL_KEY : key;
    }

    /**
     * 将null键的内部表示形式返回给调用者，返回null。
     */
    static Object unmaskNull(Object key) {
        return (key == NULL_KEY) ? null : key;
    }

    /**
     * 检查非空引用x和可能为空的y是否相等。 默认情况下使用Object.equals。
     */
    private static boolean eq(Object x, Object y) {
        return x == y || x.equals(y);
    }

    /**
     * 检索对象哈希码并将补充哈希函数应用于结果哈希，以防止质量差的哈希函数。 
     * 这很关键，因为HashMap使用两个幂的长度哈希表，否则会遇到低位不同的
     * hashCodes的冲突。
     */
    final int hash(Object k) {
        int h = k.hashCode();

        // 此函数确保在每个位位置仅由常数倍数相差的hashCodes具有有限的冲突数
        //（默认加载因子大约为8）。
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }

    /**
     * Returns index for hash code h.
     */
    private static int indexFor(int h, int length) {
        return h & (length-1);
    }

    /**
     * 从表中清除过时的条目。s
     */
    private void expungeStaleEntries() {
        for (Object x; (x = queue.poll()) != null; ) {
            synchronized (queue) {
                @SuppressWarnings("unchecked")
                    Entry<K,V> e = (Entry<K,V>) x;
                int i = indexFor(e.hash, table.length);

                Entry<K,V> prev = table[i];
                Entry<K,V> p = prev;
                while (p != null) {
                    Entry<K,V> next = p.next;
                    if (p == e) {
                        if (prev == e)
                            table[i] = next;
                        else
                            prev.next = next;
                        // Must not null out e.next;
                        // stale entries may be in use by a HashIterator
                        e.value = null; // Help GC
                        size--;
                        break;
                    }
                    prev = p;
                    p = next;
                }
            }
        }
    }

    /**
     * 首次删除过时条目后返回表。
     */
    private Entry<K,V>[] getTable() {
        expungeStaleEntries();
        return table;
    }

    /**
     * 返回此映射中键 - 值映射的数量。 
     * 此结果是快照，可能无法反映在下次尝试访问之前将被删除的未处理条目，
     * 因为它们不再被引用。
     */
    public int size() {
        if (size == 0)
            return 0;
        expungeStaleEntries();
        return size;
    }

    /**
     * 如果此映射不包含键 - 值映射，则返回true。 
     * 此结果是快照，可能无法反映在下次尝试访问之前将被删除的未处理条目，
     * 因为它们不再被引用。
     */
    public boolean isEmpty() {
        return size() == 0;
    }

    /**
     * 返回指定键映射到的值，如果此映射不包含键的映射，则返回{@code null}。
     * 
     * 更正式地说，如果此映射包含从密钥{@code k}到值{@code v}的映射，使得
     * {@code（key == null？k == null：key.equals（k））}， 然后这个方法返
     * 回{@code v}; 否则返回{@code null}。 （最多可以有一个这样的映射。）
     *
     * {@code null}的返回值不一定表示映射不包含键的映射; 地图也可能明确地将
     * 密钥映射到{@code null}。 {@link #containsKey containsKey}操作可用于区
     * 分这两种情况。
     *
     * @see #put(Object, Object)
     */
    public V get(Object key) {
        Object k = maskNull(key);
        int h = hash(k);
        Entry<K,V>[] tab = getTable();
        int index = indexFor(h, tab.length);
        Entry<K,V> e = tab[index];
        while (e != null) {
            if (e.hash == h && eq(k, e.get()))
                return e.value;
            e = e.next;
        }
        return null;
    }

    /**
     * 如果此映射包含指定键的映射，则返回true。
     *
     * @param  key   The key whose presence in this map is to be tested
     * @return <tt>true</tt> if there is a mapping for <tt>key</tt>;
     *         <tt>false</tt> otherwise
     */
    public boolean containsKey(Object key) {
        return getEntry(key) != null;
    }

    /**
     * 返回与此映射中的指定键关联的条目。 如果映射不包含此键的映射，则返回null。
     */
    Entry<K,V> getEntry(Object key) {
        Object k = maskNull(key);
        int h = hash(k);
        Entry<K,V>[] tab = getTable();
        int index = indexFor(h, tab.length);
        Entry<K,V> e = tab[index];
        while (e != null && !(e.hash == h && eq(k, e.get())))
            e = e.next;
        return e;
    }

    /**
     * 将指定的值与此映射中的指定键相关联。 如果映射先前包含此键的映射，则替换旧值。
     *
     * @param key key with which the specified value is to be associated.
     * @param value value to be associated with the specified key.
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     */
    public V put(K key, V value) {
        Object k = maskNull(key);
        int h = hash(k);
        Entry<K,V>[] tab = getTable();
        int i = indexFor(h, tab.length);

        for (Entry<K,V> e = tab[i]; e != null; e = e.next) {
            if (h == e.hash && eq(k, e.get())) {
                V oldValue = e.value;
                if (value != oldValue)
                    e.value = value;
                return oldValue;
            }
        }

        modCount++;
        Entry<K,V> e = tab[i];
        tab[i] = new Entry<>(k, value, queue, h, e);
        if (++size >= threshold)
            resize(tab.length * 2);
        return null;
    }

    /**
     * 将此映射的内容重新放入具有更大容量的新数组中。 
     * 当此映射中的键数达到其阈值时，将自动调用此方法。
     *
     * 如果当前容量为MAXIMUM_CAPACITY，则此方法不会调整映射大小，但会将阈值设
     * 置为Integer.MAX_VALUE。 这具有防止将来呼叫的效果。
     *
     * @param newCapacity the new capacity, MUST be a power of two;
     *        must be greater than current capacity unless current
     *        capacity is MAXIMUM_CAPACITY (in which case value
     *        is irrelevant).
     */
    void resize(int newCapacity) {
        Entry<K,V>[] oldTable = getTable();
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry<K,V>[] newTable = newTable(newCapacity);
        transfer(oldTable, newTable);
        table = newTable;

        /*
         * 如果忽略null元素并处理引用队列导致大量收缩，则恢复旧表。
         * 这应该是罕见的，但避免垃圾填充表的无限扩展。
         */
        if (size >= threshold / 2) {
            threshold = (int)(newCapacity * loadFactor);
        } else {
            expungeStaleEntries();
            transfer(newTable, oldTable);
            table = oldTable;
        }
    }

    /** 将所有条目从src传输到dest表 */
    private void transfer(Entry<K,V>[] src, Entry<K,V>[] dest) {
        for (int j = 0; j < src.length; ++j) {
            Entry<K,V> e = src[j];
            src[j] = null;
            while (e != null) {
                Entry<K,V> next = e.next;
                Object key = e.get();
                if (key == null) {
                    e.next = null;  // Help GC
                    e.value = null; //  "   "
                    size--;
                } else {
                    int i = indexFor(e.hash, dest.length);
                    e.next = dest[i];
                    dest[i] = e;
                }
                e = next;
            }
        }
    }

    /**
     * 将指定映射中的所有映射复制到此映射。 
     * 这些映射将替换此映射对当前位于指定映射中的任何键的任何映射。
     *
     * @param m mappings to be stored in this map.
     * @throws  NullPointerException if the specified map is null.
     */
    public void putAll(Map<? extends K, ? extends V> m) {
        int numKeysToBeAdded = m.size();
        if (numKeysToBeAdded == 0)
            return;

        /*
         * 如果要添加的映射数大于或等于阈值，则展开地图。 这是保守的; 明显的条件是
         *（m.size（）+ size）> = threshold，但如果要添加的键与此映射中已有的键重叠
         * ，则此条件可能会导致映射具有两倍的适当容量。 
         * 通过使用保守计算，我们最多可以对自己进行一次额外的调整。
         */
        if (numKeysToBeAdded > threshold) {
            int targetCapacity = (int)(numKeysToBeAdded / loadFactor + 1);
            if (targetCapacity > MAXIMUM_CAPACITY)
                targetCapacity = MAXIMUM_CAPACITY;
            int newCapacity = table.length;
            while (newCapacity < targetCapacity)
                newCapacity <<= 1;
            if (newCapacity > table.length)
                resize(newCapacity);
        }

        for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
            put(e.getKey(), e.getValue());
    }

    /**
     * 如果存在，则从此弱哈希映射中移除键的映射。 
     * 更正式地说，如果此映射包含从键k到值v的映射，使得
     * <code>（key == null？k == null：key.equals（k））</ code>，则删除该映射。 
     *（地图最多可以包含一个这样的映射。）
     *
     * 返回此映射先前与该键关联的值，如果映射不包含该键的映射，则返回null。
     * 返回值null不一定表示映射不包含键的映射; 地图也可能将键显式映射为null。
     *
     * 一旦调用返回，映射将不包含指定键的映射。
     *
     * @param key key whose mapping is to be removed from the map
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>
     */
    public V remove(Object key) {
        Object k = maskNull(key);
        int h = hash(k);
        Entry<K,V>[] tab = getTable();
        int i = indexFor(h, tab.length);
        Entry<K,V> prev = tab[i];
        Entry<K,V> e = prev;

        while (e != null) {
            Entry<K,V> next = e.next;
            if (h == e.hash && eq(k, e.get())) {
                modCount++;
                size--;
                if (prev == e)
                    tab[i] = next;
                else
                    prev.next = next;
                return e.value;
            }
            prev = e;
            e = next;
        }

        return null;
    }

    /** Special version of remove needed by Entry set */
    boolean removeMapping(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Entry<K,V>[] tab = getTable();
        Map.Entry<?,?> entry = (Map.Entry<?,?>)o;
        Object k = maskNull(entry.getKey());
        int h = hash(k);
        int i = indexFor(h, tab.length);
        Entry<K,V> prev = tab[i];
        Entry<K,V> e = prev;

        while (e != null) {
            Entry<K,V> next = e.next;
            if (h == e.hash && e.equals(entry)) {
                modCount++;
                size--;
                if (prev == e)
                    tab[i] = next;
                else
                    prev.next = next;
                return true;
            }
            prev = e;
            e = next;
        }

        return false;
    }

    /**
     * 从此映射中删除所有映射。 此调用返回后，映射将为空。
     */
    public void clear() {
        // clear out ref queue. We don't need to expunge entries
        // since table is getting cleared.
        while (queue.poll() != null)
            ;

        modCount++;
        Arrays.fill(table, null);
        size = 0;

        // 数组的分配可能导致GC，这可能导致其他条目过时。
        // 从引用队列中删除这些条目将使它们有资格进行回收。
        while (queue.poll() != null)
            ;
    }

    /**
     * 如果此映射将一个或多个键映射到指定值，则返回true。
     *
     * @param value value whose presence in this map is to be tested
     * @return <tt>true</tt> if this map maps one or more keys to the
     *         specified value
     */
    public boolean containsValue(Object value) {
        if (value==null)
            return containsNullValue();

        Entry<K,V>[] tab = getTable();
        for (int i = tab.length; i-- > 0;)
            for (Entry<K,V> e = tab[i]; e != null; e = e.next)
                if (value.equals(e.value))
                    return true;
        return false;
    }

    /**
     * 带有null参数的containsValue的特例代码
     */
    private boolean containsNullValue() {
        Entry<K,V>[] tab = getTable();
        for (int i = tab.length; i-- > 0;)
            for (Entry<K,V> e = tab[i]; e != null; e = e.next)
                if (e.value==null)
                    return true;
        return false;
    }

    /**
     * 此哈希表中的条目使用其主ref字段作为键来扩展WeakReference。
     */
    private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V> {
        V value;
        final int hash;
        Entry<K,V> next;

        /**
         * Creates new entry.
         */
        Entry(Object key, V value,
              ReferenceQueue<Object> queue,
              int hash, Entry<K,V> next) {
            super(key, queue);
            this.value = value;
            this.hash  = hash;
            this.next  = next;
        }

        @SuppressWarnings("unchecked")
        public K getKey() {
            return (K) WeakHashMap.unmaskNull(get());
        }

        public V getValue() {
            return value;
        }

        public V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            K k1 = getKey();
            Object k2 = e.getKey();
            if (k1 == k2 || (k1 != null && k1.equals(k2))) {
                V v1 = getValue();
                Object v2 = e.getValue();
                if (v1 == v2 || (v1 != null && v1.equals(v2)))
                    return true;
            }
            return false;
        }

        public int hashCode() {
            K k = getKey();
            V v = getValue();
            return Objects.hashCode(k) ^ Objects.hashCode(v);
        }

        public String toString() {
            return getKey() + "=" + getValue();
        }
    }

    private abstract class HashIterator<T> implements Iterator<T> {
        private int index;
        private Entry<K,V> entry;
        private Entry<K,V> lastReturned;
        private int expectedModCount = modCount;

        /**
         * 需要强有力的参考以避免hasNext和next之间的密钥消失
         */
        private Object nextKey;

        /**
         * 需要强引用以避免nextEntry（）和任何条目使用之间的密钥消失
         */
        private Object currentKey;

        HashIterator() {
            index = isEmpty() ? 0 : table.length;
        }

        public boolean hasNext() {
            Entry<K,V>[] t = table;

            while (nextKey == null) {
                Entry<K,V> e = entry;
                int i = index;
                while (e == null && i > 0)
                    e = t[--i];
                entry = e;
                index = i;
                if (e == null) {
                    currentKey = null;
                    return false;
                }
                nextKey = e.get(); // hold on to key in strong ref
                if (nextKey == null)
                    entry = entry.next;
            }
            return true;
        }

        /** 跨越不同类型的迭代器的next（）的公共部分 */
        protected Entry<K,V> nextEntry() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            if (nextKey == null && !hasNext())
                throw new NoSuchElementException();

            lastReturned = entry;
            entry = entry.next;
            currentKey = nextKey;
            nextKey = null;
            return lastReturned;
        }

        public void remove() {
            if (lastReturned == null)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();

            WeakHashMap.this.remove(currentKey);
            expectedModCount = modCount;
            lastReturned = null;
            currentKey = null;
        }

    }

    private class ValueIterator extends HashIterator<V> {
        public V next() {
            return nextEntry().value;
        }
    }

    private class KeyIterator extends HashIterator<K> {
        public K next() {
            return nextEntry().getKey();
        }
    }

    private class EntryIterator extends HashIterator<Map.Entry<K,V>> {
        public Map.Entry<K,V> next() {
            return nextEntry();
        }
    }

    // Views

    private transient Set<Map.Entry<K,V>> entrySet;

    /**
     * 返回此映射中包含的键的{@link Set}视图。
     * 该集由地图支持，因此对地图的更改将反映在集中，反之亦然。 如果在对集合
     * 进行迭代时修改了映射（除了通过迭代器自己的remove操作），迭代的结果是未
     * 定义的。 该集支持元素删除，它通过Iterator.remove，Set.remove，removeAll，
     * retainAll和clear操作从地图中删除相应的映射。 它不支持add或addAll操作。
     */
    public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new KeySet();
            keySet = ks;
        }
        return ks;
    }

    private class KeySet extends AbstractSet<K> {
        public Iterator<K> iterator() {
            return new KeyIterator();
        }

        public int size() {
            return WeakHashMap.this.size();
        }

        public boolean contains(Object o) {
            return containsKey(o);
        }

        public boolean remove(Object o) {
            if (containsKey(o)) {
                WeakHashMap.this.remove(o);
                return true;
            }
            else
                return false;
        }

        public void clear() {
            WeakHashMap.this.clear();
        }

        public Spliterator<K> spliterator() {
            return new KeySpliterator<>(WeakHashMap.this, 0, -1, 0, 0);
        }
    }

    /**
     * 返回此映射中包含的值的{@link Collection}视图。 
     * 该集合由地图支持，因此对地图的更改将反映在集合中，反之亦然。 如果在
     * 对集合进行迭代时修改了映射（除了通过迭代器自己的remove操作），迭代
     * 的结果是未定义的。 该集合支持元素删除，它通过Iterator.remove，
     * Collection.remove，removeAll，retainAll和clear操作从地图中删除相应
     * 的映射。 它不支持add或addAll操作。
     */
    public Collection<V> values() {
        Collection<V> vs = values;
        if (vs == null) {
            vs = new Values();
            values = vs;
        }
        return vs;
    }

    private class Values extends AbstractCollection<V> {
        public Iterator<V> iterator() {
            return new ValueIterator();
        }

        public int size() {
            return WeakHashMap.this.size();
        }

        public boolean contains(Object o) {
            return containsValue(o);
        }

        public void clear() {
            WeakHashMap.this.clear();
        }

        public Spliterator<V> spliterator() {
            return new ValueSpliterator<>(WeakHashMap.this, 0, -1, 0, 0);
        }
    }

    /**
     * 返回此映射中包含的映射的{@link Set}视图。 
     * 该集由地图支持，因此对地图的更改将反映在集中，反之亦然。 如果在对集
     * 合进行迭代时修改了映射（除非通过迭代器自己的remove操作，或者通过迭代
     * 器返回的映射条目上的setValue操作），迭代的结果是未定义的。 
     * 该集支持元素删除，它通过Iterator.remove，Set.remove，removeAll，retainAll和clear操
     * 作从地图中删除相应的映射。 它不支持add或addAll操作。
     */
    public Set<Map.Entry<K,V>> entrySet() {
        Set<Map.Entry<K,V>> es = entrySet;
        return es != null ? es : (entrySet = new EntrySet());
    }

    private class EntrySet extends AbstractSet<Map.Entry<K,V>> {
        public Iterator<Map.Entry<K,V>> iterator() {
            return new EntryIterator();
        }

        public boolean contains(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            Entry<K,V> candidate = getEntry(e.getKey());
            return candidate != null && candidate.equals(e);
        }

        public boolean remove(Object o) {
            return removeMapping(o);
        }

        public int size() {
            return WeakHashMap.this.size();
        }

        public void clear() {
            WeakHashMap.this.clear();
        }

        private List<Map.Entry<K,V>> deepCopy() {
            List<Map.Entry<K,V>> list = new ArrayList<>(size());
            for (Map.Entry<K,V> e : this)
                list.add(new AbstractMap.SimpleEntry<>(e));
            return list;
        }

        public Object[] toArray() {
            return deepCopy().toArray();
        }

        public <T> T[] toArray(T[] a) {
            return deepCopy().toArray(a);
        }

        public Spliterator<Map.Entry<K,V>> spliterator() {
            return new EntrySpliterator<>(WeakHashMap.this, 0, -1, 0, 0);
        }
    }

    @SuppressWarnings("unchecked")
    @Override
    public void forEach(BiConsumer<? super K, ? super V> action) {
        Objects.requireNonNull(action);
        int expectedModCount = modCount;

        Entry<K, V>[] tab = getTable();
        for (Entry<K, V> entry : tab) {
            while (entry != null) {
                Object key = entry.get();
                if (key != null) {
                    action.accept((K)WeakHashMap.unmaskNull(key), entry.value);
                }
                entry = entry.next;

                if (expectedModCount != modCount) {
                    throw new ConcurrentModificationException();
                }
            }
        }
    }

    @SuppressWarnings("unchecked")
    @Override
    public void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
        Objects.requireNonNull(function);
        int expectedModCount = modCount;

        Entry<K, V>[] tab = getTable();;
        for (Entry<K, V> entry : tab) {
            while (entry != null) {
                Object key = entry.get();
                if (key != null) {
                    entry.value = function.apply((K)WeakHashMap.unmaskNull(key), entry.value);
                }
                entry = entry.next;

                if (expectedModCount != modCount) {
                    throw new ConcurrentModificationException();
                }
            }
        }
    }

    /**
     * 与其他散列Spliterator类似的形式，但跳过死元素。
     */
    static class WeakHashMapSpliterator<K,V> {
        final WeakHashMap<K,V> map;
        WeakHashMap.Entry<K,V> current; // current node
        int index;             // current index, modified on advance/split
        int fence;             // -1 until first use; then one past last index
        int est;               // size estimate
        int expectedModCount;  // for comodification checks

        WeakHashMapSpliterator(WeakHashMap<K,V> m, int origin,
                               int fence, int est,
                               int expectedModCount) {
            this.map = m;
            this.index = origin;
            this.fence = fence;
            this.est = est;
            this.expectedModCount = expectedModCount;
        }

        final int getFence() { // initialize fence and size on first use
            int hi;
            if ((hi = fence) < 0) {
                WeakHashMap<K,V> m = map;
                est = m.size();
                expectedModCount = m.modCount;
                hi = fence = m.table.length;
            }
            return hi;
        }

        public final long estimateSize() {
            getFence(); // force init
            return (long) est;
        }
    }

    static final class KeySpliterator<K,V>
        extends WeakHashMapSpliterator<K,V>
        implements Spliterator<K> {
        KeySpliterator(WeakHashMap<K,V> m, int origin, int fence, int est,
                       int expectedModCount) {
            super(m, origin, fence, est, expectedModCount);
        }

        public KeySpliterator<K,V> trySplit() {
            int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
            return (lo >= mid) ? null :
                new KeySpliterator<K,V>(map, lo, index = mid, est >>>= 1,
                                        expectedModCount);
        }

        public void forEachRemaining(Consumer<? super K> action) {
            int i, hi, mc;
            if (action == null)
                throw new NullPointerException();
            WeakHashMap<K,V> m = map;
            WeakHashMap.Entry<K,V>[] tab = m.table;
            if ((hi = fence) < 0) {
                mc = expectedModCount = m.modCount;
                hi = fence = tab.length;
            }
            else
                mc = expectedModCount;
            if (tab.length >= hi && (i = index) >= 0 &&
                (i < (index = hi) || current != null)) {
                WeakHashMap.Entry<K,V> p = current;
                current = null; // exhaust
                do {
                    if (p == null)
                        p = tab[i++];
                    else {
                        Object x = p.get();
                        p = p.next;
                        if (x != null) {
                            @SuppressWarnings("unchecked") K k =
                                (K) WeakHashMap.unmaskNull(x);
                            action.accept(k);
                        }
                    }
                } while (p != null || i < hi);
            }
            if (m.modCount != mc)
                throw new ConcurrentModificationException();
        }

        public boolean tryAdvance(Consumer<? super K> action) {
            int hi;
            if (action == null)
                throw new NullPointerException();
            WeakHashMap.Entry<K,V>[] tab = map.table;
            if (tab.length >= (hi = getFence()) && index >= 0) {
                while (current != null || index < hi) {
                    if (current == null)
                        current = tab[index++];
                    else {
                        Object x = current.get();
                        current = current.next;
                        if (x != null) {
                            @SuppressWarnings("unchecked") K k =
                                (K) WeakHashMap.unmaskNull(x);
                            action.accept(k);
                            if (map.modCount != expectedModCount)
                                throw new ConcurrentModificationException();
                            return true;
                        }
                    }
                }
            }
            return false;
        }

        public int characteristics() {
            return Spliterator.DISTINCT;
        }
    }

    static final class ValueSpliterator<K,V>
        extends WeakHashMapSpliterator<K,V>
        implements Spliterator<V> {
        ValueSpliterator(WeakHashMap<K,V> m, int origin, int fence, int est,
                         int expectedModCount) {
            super(m, origin, fence, est, expectedModCount);
        }

        public ValueSpliterator<K,V> trySplit() {
            int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
            return (lo >= mid) ? null :
                new ValueSpliterator<K,V>(map, lo, index = mid, est >>>= 1,
                                          expectedModCount);
        }

        public void forEachRemaining(Consumer<? super V> action) {
            int i, hi, mc;
            if (action == null)
                throw new NullPointerException();
            WeakHashMap<K,V> m = map;
            WeakHashMap.Entry<K,V>[] tab = m.table;
            if ((hi = fence) < 0) {
                mc = expectedModCount = m.modCount;
                hi = fence = tab.length;
            }
            else
                mc = expectedModCount;
            if (tab.length >= hi && (i = index) >= 0 &&
                (i < (index = hi) || current != null)) {
                WeakHashMap.Entry<K,V> p = current;
                current = null; // exhaust
                do {
                    if (p == null)
                        p = tab[i++];
                    else {
                        Object x = p.get();
                        V v = p.value;
                        p = p.next;
                        if (x != null)
                            action.accept(v);
                    }
                } while (p != null || i < hi);
            }
            if (m.modCount != mc)
                throw new ConcurrentModificationException();
        }

        public boolean tryAdvance(Consumer<? super V> action) {
            int hi;
            if (action == null)
                throw new NullPointerException();
            WeakHashMap.Entry<K,V>[] tab = map.table;
            if (tab.length >= (hi = getFence()) && index >= 0) {
                while (current != null || index < hi) {
                    if (current == null)
                        current = tab[index++];
                    else {
                        Object x = current.get();
                        V v = current.value;
                        current = current.next;
                        if (x != null) {
                            action.accept(v);
                            if (map.modCount != expectedModCount)
                                throw new ConcurrentModificationException();
                            return true;
                        }
                    }
                }
            }
            return false;
        }

        public int characteristics() {
            return 0;
        }
    }

    static final class EntrySpliterator<K,V>
        extends WeakHashMapSpliterator<K,V>
        implements Spliterator<Map.Entry<K,V>> {
        EntrySpliterator(WeakHashMap<K,V> m, int origin, int fence, int est,
                       int expectedModCount) {
            super(m, origin, fence, est, expectedModCount);
        }

        public EntrySpliterator<K,V> trySplit() {
            int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
            return (lo >= mid) ? null :
                new EntrySpliterator<K,V>(map, lo, index = mid, est >>>= 1,
                                          expectedModCount);
        }


        public void forEachRemaining(Consumer<? super Map.Entry<K, V>> action) {
            int i, hi, mc;
            if (action == null)
                throw new NullPointerException();
            WeakHashMap<K,V> m = map;
            WeakHashMap.Entry<K,V>[] tab = m.table;
            if ((hi = fence) < 0) {
                mc = expectedModCount = m.modCount;
                hi = fence = tab.length;
            }
            else
                mc = expectedModCount;
            if (tab.length >= hi && (i = index) >= 0 &&
                (i < (index = hi) || current != null)) {
                WeakHashMap.Entry<K,V> p = current;
                current = null; // exhaust
                do {
                    if (p == null)
                        p = tab[i++];
                    else {
                        Object x = p.get();
                        V v = p.value;
                        p = p.next;
                        if (x != null) {
                            @SuppressWarnings("unchecked") K k =
                                (K) WeakHashMap.unmaskNull(x);
                            action.accept
                                (new AbstractMap.SimpleImmutableEntry<K,V>(k, v));
                        }
                    }
                } while (p != null || i < hi);
            }
            if (m.modCount != mc)
                throw new ConcurrentModificationException();
        }

        public boolean tryAdvance(Consumer<? super Map.Entry<K,V>> action) {
            int hi;
            if (action == null)
                throw new NullPointerException();
            WeakHashMap.Entry<K,V>[] tab = map.table;
            if (tab.length >= (hi = getFence()) && index >= 0) {
                while (current != null || index < hi) {
                    if (current == null)
                        current = tab[index++];
                    else {
                        Object x = current.get();
                        V v = current.value;
                        current = current.next;
                        if (x != null) {
                            @SuppressWarnings("unchecked") K k =
                                (K) WeakHashMap.unmaskNull(x);
                            action.accept
                                (new AbstractMap.SimpleImmutableEntry<K,V>(k, v));
                            if (map.modCount != expectedModCount)
                                throw new ConcurrentModificationException();
                            return true;
                        }
                    }
                }
            }
            return false;
        }

        public int characteristics() {
            return Spliterator.DISTINCT;
        }
    }

}

```