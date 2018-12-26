```java 
package java.util;

import java.lang.reflect.Array;
import java.util.function.BiConsumer;
import java.util.function.BiFunction;
import java.util.function.Consumer;
import sun.misc.SharedSecrets;

 
/**
 * 此类使用哈希表实现Map接口，在比较键（和值）时使用引用相等性代替对象相等性。
 * 换句话说，在IdentityHashMap中，当且仅当（k1 == k2）时，两个密钥k1和k2被认
 * 为是相等的。 （在正常的Map实现中（如HashMap），当且仅当
 * （k1 == null？k2 == null：k1.equals（k2））时，两个键k1和k2被认为是相等的。）
 *
 *
 *    这个类不是通用的Map实现！虽然这个类实现了Map接口，但它故意违反了Map的一
 * 般契约，它要求在比较对象时使用equals方法。此类仅用于需要引用相等语义的罕见情况。
 *
 *    此类的典型用法是保留拓扑的对象图转换，例如序列化或深度复制。要执行此类转换
 * ，程序必须维护一个“节点表”，以跟踪已处理的所有对象引用。节点表必须不等同于不同
 * 的对象，即使它们碰巧相等。此类的另一个典型用法是维护代理对象。例如，调试工具可
 * 能希望为正在调试的程序中的每个对象维护一个代理对象。
 *
 *
 *    此类提供所有可选的映射操作，并允许空值和空键。这个类不保证地图的顺序;特别是，
 * 它不保证订单会随着时间的推移保持不变。
 *
 *
 *    假设系统标识哈希函数（{@link System＃identityHashCode（Object）}）在桶之间
 * 正确地分散元素，该类为基本操作（get和put）提供恒定时间性能。
 *
 *
 *    此类有一个调整参数（影响性能但不影响语义）：预期的最大大小。此参数是地图预
 * 期保留的最大键值映射数。在内部，此参数用于确定最初包含哈希表的存储桶数。未指定
 * 预期最大大小和桶数之间的精确关系。
 *
 *
 *    如果映射的大小（键值映射的数量）充分超过预期的最大大小，则桶的数量增加。增
 * 加桶的数量（“rehashing”）可能相当昂贵，因此创建具有足够大的预期最大大小的身份
 * 哈希映射是值得的。另一方面，对集合视图的迭代需要与哈希表中的桶数成比例的时间，
 * 因此如果您特别关注迭代性能或内存使用，则不应将预期的最大大小设置得太高。
 *
 *
 *    请注意，此实现不同步。如果多个线程同时访问标识哈希映射，并且至少有一个线程
 * 在结构上修改了映射，则必须在外部进行同步。 （结构修改是添加或删除一个或多个映
 * 射的任何操作;仅更改与实例已包含的键关联的值不是结构修改。）这通常通过同步自然
 * 封装映射的某个对象来完成。 。
 *
 *
 *    如果不存在此类对象，则应使用
 * {@link Collections＃synchronizedMap Collections.synchronizedMap}方法“包装”映射。这最好在创建时完成，以防止对映射的意外不同步访
 * 问：Map m = Collections.synchronizedMap（new IdentityHashMap（...））;
 *
 *    所有这个类的“集合视图方法”返回的集合的迭代器方法返回的迭代器是快速失败的：如
 * 果在创建迭代器之后的任何时候对映射进行结构修改，除非通过迭代器自己的删除方法，
 * 迭代器将抛出{@link ConcurrentModificationException}。因此，在并发修改的情况下，
 * 迭代器快速而干净地失败，而不是在未来的未确定时间冒任意，非确定性行为的风险。
 *
 *
 *    请注意，迭代器的快速失败行为无法得到保证，因为一般来说，在存在不同步的并发修
 * 改时，不可能做出任何硬性保证。失败快速迭代器会尽最大努力抛出
 * ConcurrentModificationException。因此，编写依赖于此异常的程序以确保其正确性是错
 * 误的：故障快速迭代器应仅用于检测错误。
 *
 *    实现说明：这是一个简单的线性探测哈希表，如Sedgewick和Knuth的文本中所述。数组
 * 交替显示保持键和值。 （对于大型表，这比使用单独的数组具有更好的局部性。）对于许
 * 多JRE实现和操作混合，此类将产生比{@link HashM更好的性能
 *
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @see     System#identityHashCode(Object)
 * @see     Object#hashCode()
 * @see     Collection
 * @see     Map
 * @see     HashMap
 * @see     TreeMap
 * @author  Doug Lea and Josh Bloch
 * @since   1.4
 */

public class IdentityHashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, java.io.Serializable, Cloneable
{
    /**
     * no-args构造函数使用的初始容量。 必须是两个人的力量。 在给定载荷因子为2/3的情况下
     * ，值32对应于（指定的）预期最大尺寸21。
     */
    private static final int DEFAULT_CAPACITY = 32;

    /**
     *  最小容量，如果具有参数的任一构造函数隐式指定较低值，则使用该最小容量。
     *  在给定载荷因子为2/3的情况下，值4对应于预期的最大尺寸2。 必须是两个人的力量。
     */
    private static final int MINIMUM_CAPACITY = 4;

    /**
     * 如果具有参数的任一构造函数隐式指定较高值，则使用最大容量。 必须是两个
     * <= 1 << 29的幂。 实际上，map可以容纳不超过MAXIMUM_CAPACITY-1项，因为它必须
     * 至少有一个带键== null的槽，以避免get（），put（），remove（）中的无限循环
     */
    private static final int MAXIMUM_CAPACITY = 1 << 29;

    /**
     * 该表根据需要调整大小。 长度必须始终是2的幂。
     */
    transient Object[] table; // non-private to simplify nested class access

    /**
     *  此标识哈希映射中包含的键 - 值映射的数量。
     *
     * @serial
     */
    int size;

    /**
     * 修改次数，支持快速失败的迭代器
     */
    transient int modCount;

    /**
     * 表示表内的空键的值。
     */
    static final Object NULL_KEY = new Object();

    /**
     * 如果为null，则使用NULL_KEY作为键。
     */
    private static Object maskNull(Object key) {
        return (key == null ? NULL_KEY : key);
    }

    /**
     * 将null键的内部表示形式返回给调用者，返回null。
     */
    static final Object unmaskNull(Object key) {
        return (key == NULL_KEY ? null : key);
    }

    /**
     * 使用默认的预期最大大小构造一个新的空标识哈希映射（21）。
     */
    public IdentityHashMap() {
        init(DEFAULT_CAPACITY);
    }

    /**
     * 使用指定的预期最大大小构造一个新的空映射。 
     * 将超过预期数量的键值映射放入映射可能会导致内部数据结构增长，这可能有些耗时。
     *
     * @param expectedMaxSize the expected maximum size of the map
     * @throws IllegalArgumentException if <tt>expectedMaxSize</tt> is negative
     */
    public IdentityHashMap(int expectedMaxSize) {
        if (expectedMaxSize < 0)
            throw new IllegalArgumentException("expectedMaxSize is negative: "
                                               + expectedMaxSize);
        init(capacity(expectedMaxSize));
    }

    /**
     * 返回给定预期最大大小的适当容量。 返回MINIMUM_CAPACITY和MAXIMUM_CAPACITY之
     * 间的最小幂2，包括大于（3 * expectedMaxSize）/ 2（如果存在这样的数字）。 否
     * 则返回MAXIMUM_CAPACITY。
     */
    private static int capacity(int expectedMaxSize) {
        // assert expectedMaxSize >= 0;
        return
            (expectedMaxSize > MAXIMUM_CAPACITY / 3) ? MAXIMUM_CAPACITY :
            (expectedMaxSize <= 2 * MINIMUM_CAPACITY / 3) ? MINIMUM_CAPACITY :
            Integer.highestOneBit(expectedMaxSize + (expectedMaxSize << 1));
    }

    /**
     * 将对象初始化为具有指定初始容量的空映射，假定为MINIMUM_CAPACITY和MAXIMUM_CAPACITY
     * （包括两者）之间的2的幂。
     */
    private void init(int initCapacity) {
        // assert (initCapacity & -initCapacity) == initCapacity; // power of 2
        // assert initCapacity >= MINIMUM_CAPACITY;
        // assert initCapacity <= MAXIMUM_CAPACITY;

        table = new Object[2 * initCapacity];
    }

    /**
     * 构造一个新的标识哈希映射，其中包含指定映射中的键 - 值映射。
     *
     * @param m the map whose mappings are to be placed into this map
     * @throws NullPointerException if the specified map is null
     */
    public IdentityHashMap(Map<? extends K, ? extends V> m) {
        // Allow for a bit of growth
        this((int) ((1 + m.size()) * 1.1));
        putAll(m);
    }

    /**
     * 返回此标识哈希映射中键 - 值映射的数量。
     *
     * @return the number of key-value mappings in this map
     */
    public int size() {
        return size;
    }

    /**
     * Returns <tt>true</tt> if this identity hash map contains no key-value
     * mappings.
     *
     * @return <tt>true</tt> if this identity hash map contains no key-value
     *         mappings
     */
    public boolean isEmpty() {
        return size == 0;
    }

    /**
     * Returns index for Object x.
     */
    private static int hash(Object x, int length) {
        int h = System.identityHashCode(x);
        // Multiply by -127, and left-shift to use least bit as part of hash
        return ((h << 1) - (h << 8)) & (length - 1);
    }

    /**
     * Circularly traverses table of size len.
     */
    private static int nextKeyIndex(int i, int len) {
        return (i + 2 < len ? i + 2 : 0);
    }

    /**
     * 返回指定键映射到的值，如果此映射不包含键的映射，则返回{@code null}。更正式地说，
	 * 如果此映射包含从密钥{@code k}到值{@code v}的映射，使得{@code（key == k）}，则此
	 * 方法返回{@code v};否则返回{@code null}。 （最多可以有一个这样的映射。）{@code 
	 * null}的返回值不一定表示映射不包含密钥的映射;地图也可能明确地将密钥映射到
	 * {@code null}。 {@link #containsKey containsKey}操作可用于区分这两种情况。
     *
     * @see #put(Object, Object)
     */
    @SuppressWarnings("unchecked")
    public V get(Object key) {
        Object k = maskNull(key);
        Object[] tab = table;
        int len = tab.length;
        int i = hash(k, len);
        while (true) {
            Object item = tab[i];
            if (item == k)
                return (V) tab[i + 1];
            if (item == null)
                return null;
            i = nextKeyIndex(i, len);
        }
    }

    /**
     * Tests whether the specified object reference is a key in this identity
     * hash map.
     *
     * @param   key   possible key
     * @return  <code>true</code> if the specified object reference is a key
     *          in this map
     * @see     #containsValue(Object)
     */
    public boolean containsKey(Object key) {
        Object k = maskNull(key);
        Object[] tab = table;
        int len = tab.length;
        int i = hash(k, len);
        while (true) {
            Object item = tab[i];
            if (item == k)
                return true;
            if (item == null)
                return false;
            i = nextKeyIndex(i, len);
        }
    }

    /**
     * Tests whether the specified object reference is a value in this identity
     * hash map.
     *
     * @param value value whose presence in this map is to be tested
     * @return <tt>true</tt> if this map maps one or more keys to the
     *         specified object reference
     * @see     #containsKey(Object)
     */
    public boolean containsValue(Object value) {
        Object[] tab = table;
        for (int i = 1; i < tab.length; i += 2)
            if (tab[i] == value && tab[i - 1] != null)
                return true;

        return false;
    }

    /**
     * Tests if the specified key-value mapping is in the map.
     *
     * @param   key   possible key
     * @param   value possible value
     * @return  <code>true</code> if and only if the specified key-value
     *          mapping is in the map
     */
    private boolean containsMapping(Object key, Object value) {
        Object k = maskNull(key);
        Object[] tab = table;
        int len = tab.length;
        int i = hash(k, len);
        while (true) {
            Object item = tab[i];
            if (item == k)
                return tab[i + 1] == value;
            if (item == null)
                return false;
            i = nextKeyIndex(i, len);
        }
    }

    /**
     * 将指定的值与此标识哈希映射中的指定键相关联。如果映射先前包含键的映射，则替换旧值。
     *
     * @param key the key with which the specified value is to be associated
     * @param value the value to be associated with the specified key
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     * @see     Object#equals(Object)
     * @see     #get(Object)
     * @see     #containsKey(Object)
     */
    public V put(K key, V value) {
        final Object k = maskNull(key);

        retryAfterResize: for (;;) {
            final Object[] tab = table;
            final int len = tab.length;
            int i = hash(k, len);

            for (Object item; (item = tab[i]) != null;
                 i = nextKeyIndex(i, len)) {
                if (item == k) {
                    @SuppressWarnings("unchecked")
                        V oldValue = (V) tab[i + 1];
                    tab[i + 1] = value;
                    return oldValue;
                }
            }

            final int s = size + 1;
            // Use optimized form of 3 * s.
            // Next capacity is len, 2 * current capacity.
            if (s + (s << 1) > len && resize(len))
                continue retryAfterResize;

            modCount++;
            tab[i] = k;
            tab[i + 1] = value;
            size = s;
            return null;
        }
    }

    /**
     * Resizes the table if necessary to hold given capacity.
     *
     * @param newCapacity the new capacity, must be a power of two.
     * @return whether a resize did in fact take place
     */
    private boolean resize(int newCapacity) {
        // assert (newCapacity & -newCapacity) == newCapacity; // power of 2
        int newLength = newCapacity * 2;

        Object[] oldTable = table;
        int oldLength = oldTable.length;
        if (oldLength == 2 * MAXIMUM_CAPACITY) { // can't expand any further
            if (size == MAXIMUM_CAPACITY - 1)
                throw new IllegalStateException("Capacity exhausted.");
            return false;
        }
        if (oldLength >= newLength)
            return false;

        Object[] newTable = new Object[newLength];

        for (int j = 0; j < oldLength; j += 2) {
            Object key = oldTable[j];
            if (key != null) {
                Object value = oldTable[j+1];
                oldTable[j] = null;
                oldTable[j+1] = null;
                int i = hash(key, newLength);
                while (newTable[i] != null)
                    i = nextKeyIndex(i, newLength);
                newTable[i] = key;
                newTable[i + 1] = value;
            }
        }
        table = newTable;
        return true;
    }

    /**
     * 将指定映射中的所有映射复制到此映射。这些映射将替换此映射对当前位于指定映射中
     * 的任何键的任何映射。 
     *
     * @param m mappings to be stored in this map
     * @throws NullPointerException if the specified map is null
     */
    public void putAll(Map<? extends K, ? extends V> m) {
        int n = m.size();
        if (n == 0)
            return;
        if (n > size)
            resize(capacity(n)); // conservatively pre-expand

        for (Entry<? extends K, ? extends V> e : m.entrySet())
            put(e.getKey(), e.getValue());
    }

    /**
     * Removes the mapping for this key from this map if present.
     *
     * @param key key whose mapping is to be removed from the map
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     */
    public V remove(Object key) {
        Object k = maskNull(key);
        Object[] tab = table;
        int len = tab.length;
        int i = hash(k, len);

        while (true) {
            Object item = tab[i];
            if (item == k) {
                modCount++;
                size--;
                @SuppressWarnings("unchecked")
                    V oldValue = (V) tab[i + 1];
                tab[i + 1] = null;
                tab[i] = null;
                closeDeletion(i);
                return oldValue;
            }
            if (item == null)
                return null;
            i = nextKeyIndex(i, len);
        }
    }

    /**
     * Removes the specified key-value mapping from the map if it is present.
     *
     * @param   key   possible key
     * @param   value possible value
     * @return  <code>true</code> if and only if the specified key-value
     *          mapping was in the map
     */
    private boolean removeMapping(Object key, Object value) {
        Object k = maskNull(key);
        Object[] tab = table;
        int len = tab.length;
        int i = hash(k, len);

        while (true) {
            Object item = tab[i];
            if (item == k) {
                if (tab[i + 1] != value)
                    return false;
                modCount++;
                size--;
                tab[i] = null;
                tab[i + 1] = null;
                closeDeletion(i);
                return true;
            }
            if (item == null)
                return false;
            i = nextKeyIndex(i, len);
        }
    }

    /**
     *  删除后重新删除所有可能发生碰撞的条目。这样可以保留get，put等所需的线性探针碰撞属性。
     *
     * @param d the index of a newly empty deleted slot
     */
    private void closeDeletion(int d) {
        // Adapted from Knuth Section 6.4 Algorithm R
        Object[] tab = table;
        int len = tab.length;

        // Look for items to swap into newly vacated slot
        // starting at index immediately following deletion,
        // and continuing until a null slot is seen, indicating
        // the end of a run of possibly-colliding keys.
        Object item;
        for (int i = nextKeyIndex(d, len); (item = tab[i]) != null;
             i = nextKeyIndex(i, len) ) {
            // The following test triggers if the item at slot i (which
            // hashes to be at slot r) should take the spot vacated by d.
            // If so, we swap it in, and then continue with d now at the
            // newly vacated i.  This process will terminate when we hit
            // the null slot at the end of this run.
            // The test is messy because we are using a circular table.
            int r = hash(item, len);
            if ((i < r && (r <= d || d <= i)) || (r <= d && d <= i)) {
                tab[d] = item;
                tab[d + 1] = tab[i + 1];
                tab[i] = null;
                tab[i + 1] = null;
                d = i;
            }
        }
    }

    /**
     * Removes all of the mappings from this map.
     * The map will be empty after this call returns.
     */
    public void clear() {
        modCount++;
        Object[] tab = table;
        for (int i = 0; i < tab.length; i++)
            tab[i] = null;
        size = 0;
    }

    /**
     *  将指定对象与此映射进行比较以获得相等性。如果给定对象也是一个映射，并且两个映射表示
	 * 相同的对象引用映射，则返回true。更正式地说，当且仅当this.entrySet（）。
	 * equals（m.entrySet（））时，此映射等于另一个映射m。
	 * 由于此映射的基于引用相等的语义，如果将此映射与法线映射进行比较，则可能违反
	 * Object.equals协定的对称性和传递性要求。但是，Object.equals协定保证在IdentityHashMap
	 * 实例之间保持。
     *
     *  返回此映射的哈希码值。映射的哈希码被定义为映射的entrySet（）视图中每个条目的哈希码
 	 * 的总和。这确保了m1.equals（m2）意味着任何两个IdentityHashMap实例m1和m2的
	 * m1.hashCode（）== m2.hashCode（），这是{@link Object＃hashCode}的一般契约所要求的。
	 * 由于此映射的entrySet方法返回的集合中的Map.Entry实例的基于引用相等的语
	 * 义，如果两个对象中的一个可能违反前一段中提到的
	 * Object.hashCode的合同要求被比较的是IdentityHashMap实例，另一个是法线贴图。
     *
     * @param  o object to be compared for equality with this map
     * @return <tt>true</tt> if the specified object is equal to this map
     * @see Object#equals(Object)
     */
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        } else if (o instanceof IdentityHashMap) {
            IdentityHashMap<?,?> m = (IdentityHashMap<?,?>) o;
            if (m.size() != size)
                return false;

            Object[] tab = m.table;
            for (int i = 0; i < tab.length; i+=2) {
                Object k = tab[i];
                if (k != null && !containsMapping(k, tab[i + 1]))
                    return false;
            }
            return true;
        } else if (o instanceof Map) {
            Map<?,?> m = (Map<?,?>)o;
            return entrySet().equals(m.entrySet());
        } else {
            return false;  // o is not a Map
        }
    }

    /**
     * Returns the hash code value for this map.  The hash code of a map is
     * defined to be the sum of the hash codes of each entry in the map's
     * <tt>entrySet()</tt> view.  This ensures that <tt>m1.equals(m2)</tt>
     * implies that <tt>m1.hashCode()==m2.hashCode()</tt> for any two
     * <tt>IdentityHashMap</tt> instances <tt>m1</tt> and <tt>m2</tt>, as
     * required by the general contract of {@link Object#hashCode}.
     *
     * <p><b>Owing to the reference-equality-based semantics of the
     * <tt>Map.Entry</tt> instances in the set returned by this map's
     * <tt>entrySet</tt> method, it is possible that the contractual
     * requirement of <tt>Object.hashCode</tt> mentioned in the previous
     * paragraph will be violated if one of the two objects being compared is
     * an <tt>IdentityHashMap</tt> instance and the other is a normal map.</b>
     *
     * @return the hash code value for this map
     * @see Object#equals(Object)
     * @see #equals(Object)
     */
    public int hashCode() {
        int result = 0;
        Object[] tab = table;
        for (int i = 0; i < tab.length; i +=2) {
            Object key = tab[i];
            if (key != null) {
                Object k = unmaskNull(key);
                result += System.identityHashCode(k) ^
                          System.identityHashCode(tab[i + 1]);
            }
        }
        return result;
    }

    /**
     *  返回此标识哈希映射的浅表副本：不克隆键和值本身。 
     *
     * @return a shallow copy of this map
     */
    public Object clone() {
        try {
            IdentityHashMap<?,?> m = (IdentityHashMap<?,?>) super.clone();
            m.entrySet = null;
            m.table = table.clone();
            return m;
        } catch (CloneNotSupportedException e) {
            throw new InternalError(e);
        }
    }

    private abstract class IdentityHashMapIterator<T> implements Iterator<T> {
        int index = (size != 0 ? 0 : table.length); // current slot.
        int expectedModCount = modCount; // to support fast-fail
        int lastReturnedIndex = -1;      // to allow remove()
        boolean indexValid; // To avoid unnecessary next computation
        Object[] traversalTable = table; // reference to main table or copy

        public boolean hasNext() {
            Object[] tab = traversalTable;
            for (int i = index; i < tab.length; i+=2) {
                Object key = tab[i];
                if (key != null) {
                    index = i;
                    return indexValid = true;
                }
            }
            index = tab.length;
            return false;
        }

        protected int nextIndex() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            if (!indexValid && !hasNext())
                throw new NoSuchElementException();

            indexValid = false;
            lastReturnedIndex = index;
            index += 2;
            return lastReturnedIndex;
        }

        public void remove() {
            if (lastReturnedIndex == -1)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();

            expectedModCount = ++modCount;
            int deletedSlot = lastReturnedIndex;
            lastReturnedIndex = -1;
            // back up index to revisit new contents after deletion
            index = deletedSlot;
            indexValid = false;

			 //删除代码在closeDeletion中继续执行，除了
			 //它必须捕获一个罕见的情况，其中已经
			 //看到的元素被交换到一个空槽中，稍后将被这个迭代器遍历
			 //遍历。我们不能允许将来的
			 // next（）调用再次返回它。 
			 //这种情况在2/3负载因子下发生的可能性非常小，但是当它确实发生时，我们必须复
			 //制表的其余部分以用于其余的遍历。因为//这只能在我们接近表格的末尾时发生，
			 //即使在极少数情况下，这在时间或空间上也不是很昂贵。 
			 //如果我们要将一个已经看过的元素//交换到一个稍后可能被next（）返回的插槽中，
			//然后克隆表的其余部分以供将来
			// next（）调用使用。我们的副本可以在“错误”的位置有一个间隙，因为它永远不会被用于搜索。

            Object[] tab = traversalTable;
            int len = tab.length;

            int d = deletedSlot;
            Object key = tab[d];
            tab[d] = null;        // vacate the slot
            tab[d + 1] = null;

            // If traversing a copy, remove in real table.
            // We can skip gap-closure on copy.
            if (tab != IdentityHashMap.this.table) {
                IdentityHashMap.this.remove(key);
                expectedModCount = modCount;
                return;
            }

            size--;

            Object item;
            for (int i = nextKeyIndex(d, len); (item = tab[i]) != null;
                 i = nextKeyIndex(i, len)) {
                int r = hash(item, len);
                // See closeDeletion for explanation of this conditional
                if ((i < r && (r <= d || d <= i)) ||
                    (r <= d && d <= i)) {

                    // If we are about to swap an already-seen element
                    // into a slot that may later be returned by next(),
                    // then clone the rest of table for use in future
                    // next() calls. It is OK that our copy will have
                    // a gap in the "wrong" place, since it will never
                    // be used for searching anyway.

                    if (i < deletedSlot && d >= deletedSlot &&
                        traversalTable == IdentityHashMap.this.table) {
                        int remaining = len - deletedSlot;
                        Object[] newTable = new Object[remaining];
                        System.arraycopy(tab, deletedSlot,
                                         newTable, 0, remaining);
                        traversalTable = newTable;
                        index = 0;
                    }

                    tab[d] = item;
                    tab[d + 1] = tab[i + 1];
                    tab[i] = null;
                    tab[i + 1] = null;
                    d = i;
                }
            }
        }
    }

    private class KeyIterator extends IdentityHashMapIterator<K> {
        @SuppressWarnings("unchecked")
        public K next() {
            return (K) unmaskNull(traversalTable[nextIndex()]);
        }
    }

    private class ValueIterator extends IdentityHashMapIterator<V> {
        @SuppressWarnings("unchecked")
        public V next() {
            return (V) traversalTable[nextIndex() + 1];
        }
    }

    private class EntryIterator
        extends IdentityHashMapIterator<Map.Entry<K,V>>
    {
        private Entry lastReturnedEntry;

        public Map.Entry<K,V> next() {
            lastReturnedEntry = new Entry(nextIndex());
            return lastReturnedEntry;
        }

        public void remove() {
            lastReturnedIndex =
                ((null == lastReturnedEntry) ? -1 : lastReturnedEntry.index);
            super.remove();
            lastReturnedEntry.index = lastReturnedIndex;
            lastReturnedEntry = null;
        }

        private class Entry implements Map.Entry<K,V> {
            private int index;

            private Entry(int index) {
                this.index = index;
            }

            @SuppressWarnings("unchecked")
            public K getKey() {
                checkIndexForEntryUse();
                return (K) unmaskNull(traversalTable[index]);
            }

            @SuppressWarnings("unchecked")
            public V getValue() {
                checkIndexForEntryUse();
                return (V) traversalTable[index+1];
            }

            @SuppressWarnings("unchecked")
            public V setValue(V value) {
                checkIndexForEntryUse();
                V oldValue = (V) traversalTable[index+1];
                traversalTable[index+1] = value;
                // if shadowing, force into main table
                if (traversalTable != IdentityHashMap.this.table)
                    put((K) traversalTable[index], value);
                return oldValue;
            }

            public boolean equals(Object o) {
                if (index < 0)
                    return super.equals(o);

                if (!(o instanceof Map.Entry))
                    return false;
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                return (e.getKey() == unmaskNull(traversalTable[index]) &&
                       e.getValue() == traversalTable[index+1]);
            }

            public int hashCode() {
                if (lastReturnedIndex < 0)
                    return super.hashCode();

                return (System.identityHashCode(unmaskNull(traversalTable[index])) ^
                       System.identityHashCode(traversalTable[index+1]));
            }

            public String toString() {
                if (index < 0)
                    return super.toString();

                return (unmaskNull(traversalTable[index]) + "="
                        + traversalTable[index+1]);
            }

            private void checkIndexForEntryUse() {
                if (index < 0)
                    throw new IllegalStateException("Entry was removed");
            }
        }
    }

    // Views

    /**
     * This field is initialized to contain an instance of the entry set
     * view the first time this view is requested.  The view is stateless,
     * so there's no reason to create more than one.
     */
    private transient Set<Map.Entry<K,V>> entrySet;

    /**
      * 返回此映射中包含的键的基于标识的集视图。该集由地图支持，因此对地图的更改将反映在集中，
	  * 反之亦然。如果在对集合进行迭代时修改了映射，则迭代的结果是未定义的。该集支持元素删除，
	  * 它通过Iterator.remove，Set.remove，removeAll，retainAll和clear方法从地图中
	  * 删除相应的映射。它不支持add或addAll方法。
	  * 虽然此方法返回的对象实现了Set接口，但它不遵守Set的常规协定。与其支持映射一样，此方法返
	  * 回的集合将元素相等性定义为引用相等而不是对象相等。这会影响其contains，remove，
	  * containsAll，equals和hashCode方法的行为。
	  * 仅当指定对象是包含与返回集完全相同的对象引用的集时，返回集的equals方法才返回true。如
	  * 果将此方法返回的集与正常集进行比较，则可能违反Object.equals协定的对称性和传递性要求。
	  * 但是，Object.equals协定保证在此方法返回的集合中保留。
	  * 返回集的hashCode方法返回集合中元素的标识哈希码的总和，而不是其哈希码的总和。这是通过更
	  * 改equals方法的语义来强制执行的，以便在此方法返回的集合中强制执行Object.hashCode方法的
	 * 常规协定。 
     *
     * @return an identity-based set view of the keys contained in this map
     * @see Object#equals(Object)
     * @see System#identityHashCode(Object)
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
            return size;
        }
        public boolean contains(Object o) {
            return containsKey(o);
        }
        public boolean remove(Object o) {
            int oldSize = size;
            IdentityHashMap.this.remove(o);
            return size != oldSize;
        }
        /*
         * 必须从AbstractSet的impl恢复为AbstractCollection，因为前者包含一个优化，
         * 当c是较小的“普通”（非基于身份的）Set时，会导致错误的行为。
         */
        public boolean removeAll(Collection<?> c) {
            Objects.requireNonNull(c);
            boolean modified = false;
            for (Iterator<K> i = iterator(); i.hasNext(); ) {
                if (c.contains(i.next())) {
                    i.remove();
                    modified = true;
                }
            }
            return modified;
        }
        public void clear() {
            IdentityHashMap.this.clear();
        }
        public int hashCode() {
            int result = 0;
            for (K key : this)
                result += System.identityHashCode(key);
            return result;
        }
        public Object[] toArray() {
            return toArray(new Object[0]);
        }
        @SuppressWarnings("unchecked")
        public <T> T[] toArray(T[] a) {
            int expectedModCount = modCount;
            int size = size();
            if (a.length < size)
                a = (T[]) Array.newInstance(a.getClass().getComponentType(), size);
            Object[] tab = table;
            int ti = 0;
            for (int si = 0; si < tab.length; si += 2) {
                Object key;
                if ((key = tab[si]) != null) { // key present ?
                    // more elements than expected -> concurrent modification from other thread
                    if (ti >= size) {
                        throw new ConcurrentModificationException();
                    }
                    a[ti++] = (T) unmaskNull(key); // unmask key
                }
            }
            // fewer elements than expected or concurrent modification from other thread detected
            if (ti < size || expectedModCount != modCount) {
                throw new ConcurrentModificationException();
            }
            // final null marker as per spec
            if (ti < a.length) {
                a[ti] = null;
            }
            return a;
        }

        public Spliterator<K> spliterator() {
            return new KeySpliterator<>(IdentityHashMap.this, 0, -1, 0, 0);
        }
    }

    /**
	* 返回此映射中包含的值的{@link Collection}视图。该集合由地图支持，因此
	* 对地图的更改将反映在集合中，反之亦然。如果在对集合进行迭代时修改了映
	* 射，则迭代的结果是未定义的。该集合支持元素删除，它通过
	* Iterator.remove，Collection.remove，removeAll，retainAll和clear
	* 方法从地图中删除相应的映射。它不支持add或addAll方法。
	* 
	*     虽然此方法返回的对象实现了Collection接口，但它不遵循Collection的
	* 常规协定。与其支持映射一样，此方法返回的集合将元素相等性定义为引用相
	* 等而不是对象相等。这会影响其contains，remove和containsAll方法的行为。
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
            return size;
        }
        public boolean contains(Object o) {
            return containsValue(o);
        }
        public boolean remove(Object o) {
            for (Iterator<V> i = iterator(); i.hasNext(); ) {
                if (i.next() == o) {
                    i.remove();
                    return true;
                }
            }
            return false;
        }
        public void clear() {
            IdentityHashMap.this.clear();
        }
        public Object[] toArray() {
            return toArray(new Object[0]);
        }
        @SuppressWarnings("unchecked")
        public <T> T[] toArray(T[] a) {
            int expectedModCount = modCount;
            int size = size();
            if (a.length < size)
                a = (T[]) Array.newInstance(a.getClass().getComponentType(), size);
            Object[] tab = table;
            int ti = 0;
            for (int si = 0; si < tab.length; si += 2) {
                if (tab[si] != null) { // key present ?
                    // more elements than expected -> concurrent modification from other thread
                    if (ti >= size) {
                        throw new ConcurrentModificationException();
                    }
                    a[ti++] = (T) tab[si+1]; // copy value
                }
            }
            // fewer elements than expected or concurrent modification from other thread detected
            if (ti < size || expectedModCount != modCount) {
                throw new ConcurrentModificationException();
            }
            // final null marker as per spec
            if (ti < a.length) {
                a[ti] = null;
            }
            return a;
        }

        public Spliterator<V> spliterator() {
            return new ValueSpliterator<>(IdentityHashMap.this, 0, -1, 0, 0);
        }
    }

    /**
      * 返回此映射中包含的映射的{@link Set}视图。返回集合中的每个元素都是
		* 基于引用相等的Map.Entry。该集由地图支持，因此对地图的更改将反映在集中
		* ，反之亦然。如果在对集合进行迭代时修改了映射，则迭代的结果是未定义的。
		* 该集支持元素删除，它通过Iterator.remove，Set.remove，removeAll，retainAll和clear
		* 方法从地图中删除相应的映射。它不支持add或addAll方法。
		* 
		*     与支持映射一样，此方法返回的集合中的Map.Entry对象将键和值相等定
		* 义为引用相等而不是对象相等。这会影响这些Map.Entry对象的equals和
		* hashCode方法的行为。基于引用相等的Map.Entry e等于对象o当且仅当o是
		* Map.Entry和e.getKey（）== o.getKey（）
		* ＆amp;＆amp; e.getValue（）== o.getValue（）。为了适应这些等于语义，
		* hashCode方法返回System.identityHashCode（e.getKey（））^ 
		* System.identityHashCode（e.getValue（））。
		* 
		 *     由于此方法返回的集合中Map.Entry实例的基于引用相等的语义，
		* {@link Object＃equals（Object）}契约的对称性和传递性要求可能会被
		* 违反（如果有的话）将该集合中的条目与正常映射条目进行比较，或者将
		* 此方法返回的集合与一组正常映射条目（例如通过在法线图上调用此方法
		* 返回）进行比较。但是，Object.equals协定保证在基于身份的映射条目之
		* 间以及这些条目的集合中保持。
     *
     * @return a set view of the identity-mappings contained in this map
     */
    public Set<Map.Entry<K,V>> entrySet() {
        Set<Map.Entry<K,V>> es = entrySet;
        if (es != null)
            return es;
        else
            return entrySet = new EntrySet();
    }

    private class EntrySet extends AbstractSet<Map.Entry<K,V>> {
        public Iterator<Map.Entry<K,V>> iterator() {
            return new EntryIterator();
        }
        public boolean contains(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> entry = (Map.Entry<?,?>)o;
            return containsMapping(entry.getKey(), entry.getValue());
        }
        public boolean remove(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> entry = (Map.Entry<?,?>)o;
            return removeMapping(entry.getKey(), entry.getValue());
        }
        public int size() {
            return size;
        }
        public void clear() {
            IdentityHashMap.this.clear();
        }
        /*
         * Must revert from AbstractSet's impl to AbstractCollection's, as
         * the former contains an optimization that results in incorrect
         * behavior when c is a smaller "normal" (non-identity-based) Set.
         */
        public boolean removeAll(Collection<?> c) {
            Objects.requireNonNull(c);
            boolean modified = false;
            for (Iterator<Map.Entry<K,V>> i = iterator(); i.hasNext(); ) {
                if (c.contains(i.next())) {
                    i.remove();
                    modified = true;
                }
            }
            return modified;
        }

        public Object[] toArray() {
            return toArray(new Object[0]);
        }

        @SuppressWarnings("unchecked")
        public <T> T[] toArray(T[] a) {
            int expectedModCount = modCount;
            int size = size();
            if (a.length < size)
                a = (T[]) Array.newInstance(a.getClass().getComponentType(), size);
            Object[] tab = table;
            int ti = 0;
            for (int si = 0; si < tab.length; si += 2) {
                Object key;
                if ((key = tab[si]) != null) { // key present ?
                    // more elements than expected -> concurrent modification from other thread
                    if (ti >= size) {
                        throw new ConcurrentModificationException();
                    }
                    a[ti++] = (T) new AbstractMap.SimpleEntry<>(unmaskNull(key), tab[si + 1]);
                }
            }
            // fewer elements than expected or concurrent modification from other thread detected
            if (ti < size || expectedModCount != modCount) {
                throw new ConcurrentModificationException();
            }
            // final null marker as per spec
            if (ti < a.length) {
                a[ti] = null;
            }
            return a;
        }

        public Spliterator<Map.Entry<K,V>> spliterator() {
            return new EntrySpliterator<>(IdentityHashMap.this, 0, -1, 0, 0);
        }
    }


    private static final long serialVersionUID = 8188218128353913216L;

    /**
     * Saves the state of the <tt>IdentityHashMap</tt> instance to a stream
     * (i.e., serializes it).
     *
     * @serialData The <i>size</i> of the HashMap (the number of key-value
     *          mappings) (<tt>int</tt>), followed by the key (Object) and
     *          value (Object) for each key-value mapping represented by the
     *          IdentityHashMap.  The key-value mappings are emitted in no
     *          particular order.
     */
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException  {
        // Write out and any hidden stuff
        s.defaultWriteObject();

        // Write out size (number of Mappings)
        s.writeInt(size);

        // Write out keys and values (alternating)
        Object[] tab = table;
        for (int i = 0; i < tab.length; i += 2) {
            Object key = tab[i];
            if (key != null) {
                s.writeObject(unmaskNull(key));
                s.writeObject(tab[i + 1]);
            }
        }
    }

    /**
     * Reconstitutes the <tt>IdentityHashMap</tt> instance from a stream (i.e.,
     * deserializes it).
     */
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException  {
        // Read in any hidden stuff
        s.defaultReadObject();

        // Read in size (number of Mappings)
        int size = s.readInt();
        if (size < 0)
            throw new java.io.StreamCorruptedException
                ("Illegal mappings count: " + size);
        int cap = capacity(size);
        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, cap);
        init(cap);

        // Read the keys and values, and put the mappings in the table
        for (int i=0; i<size; i++) {
            @SuppressWarnings("unchecked")
                K key = (K) s.readObject();
            @SuppressWarnings("unchecked")
                V value = (V) s.readObject();
            putForCreate(key, value);
        }
    }

    /**
     * The put method for readObject.  It does not resize the table,
     * update modCount, etc.
     */
    private void putForCreate(K key, V value)
        throws java.io.StreamCorruptedException
    {
        Object k = maskNull(key);
        Object[] tab = table;
        int len = tab.length;
        int i = hash(k, len);

        Object item;
        while ( (item = tab[i]) != null) {
            if (item == k)
                throw new java.io.StreamCorruptedException();
            i = nextKeyIndex(i, len);
        }
        tab[i] = k;
        tab[i + 1] = value;
    }

    @SuppressWarnings("unchecked")
    @Override
    public void forEach(BiConsumer<? super K, ? super V> action) {
        Objects.requireNonNull(action);
        int expectedModCount = modCount;

        Object[] t = table;
        for (int index = 0; index < t.length; index += 2) {
            Object k = t[index];
            if (k != null) {
                action.accept((K) unmaskNull(k), (V) t[index + 1]);
            }

            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
        }
    }

    @SuppressWarnings("unchecked")
    @Override
    public void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
        Objects.requireNonNull(function);
        int expectedModCount = modCount;

        Object[] t = table;
        for (int index = 0; index < t.length; index += 2) {
            Object k = t[index];
            if (k != null) {
                t[index + 1] = function.apply((K) unmaskNull(k), (V) t[index + 1]);
            }

            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
        }
    }

    /**
     * Similar form as array-based Spliterators, but skips blank elements,
     * and guestimates size as decreasing by half per split.
     */
    static class IdentityHashMapSpliterator<K,V> {
        final IdentityHashMap<K,V> map;
        int index;             // current index, modified on advance/split
        int fence;             // -1 until first use; then one past last index
        int est;               // size estimate
        int expectedModCount;  // initialized when fence set

        IdentityHashMapSpliterator(IdentityHashMap<K,V> map, int origin,
                                   int fence, int est, int expectedModCount) {
            this.map = map;
            this.index = origin;
            this.fence = fence;
            this.est = est;
            this.expectedModCount = expectedModCount;
        }

        final int getFence() { // initialize fence and size on first use
            int hi;
            if ((hi = fence) < 0) {
                est = map.size;
                expectedModCount = map.modCount;
                hi = fence = map.table.length;
            }
            return hi;
        }

        public final long estimateSize() {
            getFence(); // force init
            return (long) est;
        }
    }

    static final class KeySpliterator<K,V>
        extends IdentityHashMapSpliterator<K,V>
        implements Spliterator<K> {
        KeySpliterator(IdentityHashMap<K,V> map, int origin, int fence, int est,
                       int expectedModCount) {
            super(map, origin, fence, est, expectedModCount);
        }

        public KeySpliterator<K,V> trySplit() {
            int hi = getFence(), lo = index, mid = ((lo + hi) >>> 1) & ~1;
            return (lo >= mid) ? null :
                new KeySpliterator<K,V>(map, lo, index = mid, est >>>= 1,
                                        expectedModCount);
        }

        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super K> action) {
            if (action == null)
                throw new NullPointerException();
            int i, hi, mc; Object key;
            IdentityHashMap<K,V> m; Object[] a;
            if ((m = map) != null && (a = m.table) != null &&
                (i = index) >= 0 && (index = hi = getFence()) <= a.length) {
                for (; i < hi; i += 2) {
                    if ((key = a[i]) != null)
                        action.accept((K)unmaskNull(key));
                }
                if (m.modCount == expectedModCount)
                    return;
            }
            throw new ConcurrentModificationException();
        }

        @SuppressWarnings("unchecked")
        public boolean tryAdvance(Consumer<? super K> action) {
            if (action == null)
                throw new NullPointerException();
            Object[] a = map.table;
            int hi = getFence();
            while (index < hi) {
                Object key = a[index];
                index += 2;
                if (key != null) {
                    action.accept((K)unmaskNull(key));
                    if (map.modCount != expectedModCount)
                        throw new ConcurrentModificationException();
                    return true;
                }
            }
            return false;
        }

        public int characteristics() {
            return (fence < 0 || est == map.size ? SIZED : 0) | Spliterator.DISTINCT;
        }
    }

    static final class ValueSpliterator<K,V>
        extends IdentityHashMapSpliterator<K,V>
        implements Spliterator<V> {
        ValueSpliterator(IdentityHashMap<K,V> m, int origin, int fence, int est,
                         int expectedModCount) {
            super(m, origin, fence, est, expectedModCount);
        }

        public ValueSpliterator<K,V> trySplit() {
            int hi = getFence(), lo = index, mid = ((lo + hi) >>> 1) & ~1;
            return (lo >= mid) ? null :
                new ValueSpliterator<K,V>(map, lo, index = mid, est >>>= 1,
                                          expectedModCount);
        }

        public void forEachRemaining(Consumer<? super V> action) {
            if (action == null)
                throw new NullPointerException();
            int i, hi, mc;
            IdentityHashMap<K,V> m; Object[] a;
            if ((m = map) != null && (a = m.table) != null &&
                (i = index) >= 0 && (index = hi = getFence()) <= a.length) {
                for (; i < hi; i += 2) {
                    if (a[i] != null) {
                        @SuppressWarnings("unchecked") V v = (V)a[i+1];
                        action.accept(v);
                    }
                }
                if (m.modCount == expectedModCount)
                    return;
            }
            throw new ConcurrentModificationException();
        }

        public boolean tryAdvance(Consumer<? super V> action) {
            if (action == null)
                throw new NullPointerException();
            Object[] a = map.table;
            int hi = getFence();
            while (index < hi) {
                Object key = a[index];
                @SuppressWarnings("unchecked") V v = (V)a[index+1];
                index += 2;
                if (key != null) {
                    action.accept(v);
                    if (map.modCount != expectedModCount)
                        throw new ConcurrentModificationException();
                    return true;
                }
            }
            return false;
        }

        public int characteristics() {
            return (fence < 0 || est == map.size ? SIZED : 0);
        }

    }

    static final class EntrySpliterator<K,V>
        extends IdentityHashMapSpliterator<K,V>
        implements Spliterator<Map.Entry<K,V>> {
        EntrySpliterator(IdentityHashMap<K,V> m, int origin, int fence, int est,
                         int expectedModCount) {
            super(m, origin, fence, est, expectedModCount);
        }

        public EntrySpliterator<K,V> trySplit() {
            int hi = getFence(), lo = index, mid = ((lo + hi) >>> 1) & ~1;
            return (lo >= mid) ? null :
                new EntrySpliterator<K,V>(map, lo, index = mid, est >>>= 1,
                                          expectedModCount);
        }

        public void forEachRemaining(Consumer<? super Map.Entry<K, V>> action) {
            if (action == null)
                throw new NullPointerException();
            int i, hi, mc;
            IdentityHashMap<K,V> m; Object[] a;
            if ((m = map) != null && (a = m.table) != null &&
                (i = index) >= 0 && (index = hi = getFence()) <= a.length) {
                for (; i < hi; i += 2) {
                    Object key = a[i];
                    if (key != null) {
                        @SuppressWarnings("unchecked") K k =
                            (K)unmaskNull(key);
                        @SuppressWarnings("unchecked") V v = (V)a[i+1];
                        action.accept
                            (new AbstractMap.SimpleImmutableEntry<K,V>(k, v));

                    }
                }
                if (m.modCount == expectedModCount)
                    return;
            }
            throw new ConcurrentModificationException();
        }

        public boolean tryAdvance(Consumer<? super Map.Entry<K,V>> action) {
            if (action == null)
                throw new NullPointerException();
            Object[] a = map.table;
            int hi = getFence();
            while (index < hi) {
                Object key = a[index];
                @SuppressWarnings("unchecked") V v = (V)a[index+1];
                index += 2;
                if (key != null) {
                    @SuppressWarnings("unchecked") K k =
                        (K)unmaskNull(key);
                    action.accept
                        (new AbstractMap.SimpleImmutableEntry<K,V>(k, v));
                    if (map.modCount != expectedModCount)
                        throw new ConcurrentModificationException();
                    return true;
                }
            }
            return false;
        }

        public int characteristics() {
            return (fence < 0 || est == map.size ? SIZED : 0) | Spliterator.DISTINCT;
        }
    }

}

```