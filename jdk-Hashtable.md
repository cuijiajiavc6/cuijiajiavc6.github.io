```java 

package java.util;

import java.io.*;
import java.util.concurrent.ThreadLocalRandom;
import java.util.function.BiConsumer;
import java.util.function.Function;
import java.util.function.BiFunction;
import sun.misc.SharedSecrets;

/**
 * 该类实现了一个哈希表，它将键映射到值。 任何非null对象都可以用作键或值。
 *
 * 要成功存储和检索哈希表中的对象，用作键的对象必须实现hashCode方法和equals方法。
 *
 * Hashtable的一个实例有两个影响其性能的参数：初始容量和负载因子。 
 * 容量是哈希表中的桶数，初始容量只是创建哈希表时的容量。
 * 请注意，哈希表是打开的：在“哈希冲突”的情况下，单个存储
 * 桶存储多个条目，必须按顺序搜索。 加载因子是在
 * 自动增加容量之前允许哈希表获取的完整程度的度量。
 * 初始容量和负载系数参数仅仅是实现的提示。 
 * 关于何时以及是否调用rehash方法的确切细节是依赖于实现的。
 *
 * 通常，默认负载系数（.75）在时间和空间成本之间提供了良好的折衷。 较高
 * 的值会减少空间开销，但会增加查找条目的时间成本（这反映在大多数
 * Hashtable操作中，包括ge和put）。
 *
 * 初始容量控制了浪费空间和重新运算操作的需要之间的权衡，这是非常耗时的。
 * 如果初始容量大于Hashtable将包含的最大条目数除以其加载因子，则不会发生
 * 重复操作。 但是，将初始容量设置得太高会浪费空间。
 *
 * 如果要将多个条目设置为Hashtable，则以足够大的容量创建条目可以允许更有
 * 效地插入条目，而不是根据需要执行自动重新分组来扩展表。
 *
 * 此示例创建数字哈希表。 它使用数字的名称作为键：
 * <pre>   {@code
 *   Hashtable<String, Integer> numbers
 *     = new Hashtable<String, Integer>();
 *   numbers.put("one", 1);
 *   numbers.put("two", 2);
 *   numbers.put("three", 3);}</pre>
 *
 * <p>To retrieve a number, use the following code:
 * <pre>   {@code
 *   Integer n = numbers.get("two");
 *   if (n != null) {
 *     System.out.println("two = " + n);
 *   }}</pre>
 *
 * 由所有类的“集合视图方法”返回的集合的迭代器方法返回的迭代器是快速失败的：
 * 如果在创建迭代器之后的任何时候对Hashtable进行结构修改，除非通过迭代器自
 * 己的删除 方法，迭代器将抛出{@link ConcurrentModificationException}。 因
 * 此，在并发修改的情况下，迭代器快速而干净地失败，而不是在未来的未确定时
 * 间冒任意，非确定性行为的风险。 Hashtable的键和元素方法返回的枚举不是快速
 * 失败的。
 *
 * 请注意，迭代器的快速失败行为无法得到保证，因为一般来说，在存在不同步的并
 * 发修改时，不可能做出任何硬性保证。 失败快速迭代器会尽最大努力抛出
 * ConcurrentModificationException。 
 * 因此，编写依赖于此异常的程序以确保其正确性是错误的：迭代器的快速失
 * 败行为应该仅用于检测错误。
 *
 * 从Java 2平台v1.2开始，这个类被改进以实现{@link Map}接口，使其成为
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 *
 * Java Collections Framework。 与新的集合实现不同，{@code Hashtable}是同步的。 
 * 如果不需要线程安全实现，建议使用{@link HashMap}代替{@code Hashtable}。 
 * 如果需要线程安全的高度并发实现，则建议使用
 * {@link java.util.concurrent.ConcurrentHashMap}代替{@code Hashtable}。
 *
 * @author  Arthur van Hoff
 * @author  Josh Bloch
 * @author  Neal Gafter
 * @see     Object#equals(java.lang.Object)
 * @see     Object#hashCode()
 * @see     Hashtable#rehash()
 * @see     Collection
 * @see     Map
 * @see     HashMap
 * @see     TreeMap
 * @since JDK1.0
 */
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {

    /**
     * 哈希表数据。
     */
    private transient Entry<?,?>[] table;

    /**
     * 哈希表中的条目总数。
     */
    private transient int count;

    /**
     * 当表的大小超过此阈值时，表将重新进行。 （该字段的值为（int）
     * （capacity loadFactor）。）
     *
     * @serial
     */
    private int threshold;

    /**
     * 哈希表的加载因子。
     *
     * @serial
     */
    private float loadFactor;

    /**
     * 此Hashtable已被结构修改的次数结构修改是指更改Hashtable中的条目数或
     * 以其他方式修改其内部结构（例如，重新散列）的修改。 此字段用于在
     * Hashtable的Collection-views上快速生成迭代器。
     * （请参阅ConcurrentModificationException）。
     */
    private transient int modCount = 0;

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = 1421746759512286392L;

    /**
     * 使用指定的初始容量和指定的加载因子构造一个新的空哈希表。
     *
     * @param      initialCapacity   the initial capacity of the hashtable.
     * @param      loadFactor        the load factor of the hashtable.
     * @exception  IllegalArgumentException  if the initial capacity is less
     *             than zero, or if the load factor is nonpositive.
     */
    public Hashtable(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal Load: "+loadFactor);

        if (initialCapacity==0)
            initialCapacity = 1;
        this.loadFactor = loadFactor;
        table = new Entry<?,?>[initialCapacity];
        threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
    }

    /**
     * 使用指定的初始容量和默认加载因子（0.75）构造一个新的空哈希表。
     *
     * @param     initialCapacity   the initial capacity of the hashtable.
     * @exception IllegalArgumentException if the initial capacity is less
     *              than zero.
     */
    public Hashtable(int initialCapacity) {
        this(initialCapacity, 0.75f);
    }

    /**
     * 使用默认初始容量（11）和加载因子（0.75）构造一个新的空哈希表。
     */
    public Hashtable() {
        this(11, 0.75f);
    }

    /**
     * 构造一个新的哈希表，其具有与给定Map相同的映射。 
     * 创建哈希表的初始容量足以容纳给定Map中的映射和默认加载因子（0.75）。
     *
     * @param t the map whose mappings are to be placed in this map.
     * @throws NullPointerException if the specified map is null.
     * @since   1.2
     */
    public Hashtable(Map<? extends K, ? extends V> t) {
        this(Math.max(2*t.size(), 11), 0.75f);
        putAll(t);
    }

    /**
     * 返回此哈希表中的键数。
     *
     * @return  the number of keys in this hashtable.
     */
    public synchronized int size() {
        return count;
    }

    /**
     * 测试此哈希表是否将键映射到值。
     *
     * @return  <code>true</code> if this hashtable maps no keys to values;
     *          <code>false</code> otherwise.
     */
    public synchronized boolean isEmpty() {
        return count == 0;
    }

    /**
     * 返回此哈希表中键的枚举。
     *
     * @return  an enumeration of the keys in this hashtable.
     * @see     Enumeration
     * @see     #elements()
     * @see     #keySet()
     * @see     Map
     */
    public synchronized Enumeration<K> keys() {
        return this.<K>getEnumeration(KEYS);
    }

    /**
     * 返回此哈希表中值的枚举。 在返回的对象上使用Enumeration方法按顺序获取元素。
     *
     * @return  an enumeration of the values in this hashtable.
     * @see     java.util.Enumeration
     * @see     #keys()
     * @see     #values()
     * @see     Map
     */
    public synchronized Enumeration<V> elements() {
        return this.<V>getEnumeration(VALUES);
    }

    /**
     * 测试某些键是否映射到此哈希表中的指定值。 此操作比
     * {@link #containsKey containsKey}方法更昂贵。
     *
     * 请注意，此方法的功能与{@link #containsValue containsValue}（它是集
     * 合框架中{@link Map}接口的一部分）相同。
     *
     * @param      value   a value to search for
     * @return     <code>true</code> if and only if some key maps to the
     *             <code>value</code> argument in this hashtable as
     *             determined by the <tt>equals</tt> method;
     *             <code>false</code> otherwise.
     * @exception  NullPointerException  if the value is <code>null</code>
     */
    public synchronized boolean contains(Object value) {
        if (value == null) {
            throw new NullPointerException();
        }

        Entry<?,?> tab[] = table;
        for (int i = tab.length ; i-- > 0 ;) {
            for (Entry<?,?> e = tab[i] ; e != null ; e = e.next) {
                if (e.value.equals(value)) {
                    return true;
                }
            }
        }
        return false;
    }

    /**
     * 如果此哈希表将一个或多个键映射到此值，则返回true。
     *
     * 请注意，此方法的功能与{@link #contains contains}（早于{@link Map}接口之前）相同。
     *
     * @param value value whose presence in this hashtable is to be tested
     * @return <tt>true</tt> if this map maps one or more keys to the
     *         specified value
     * @throws NullPointerException  if the value is <code>null</code>
     * @since 1.2
     */
    public boolean containsValue(Object value) {
        return contains(value);
    }

    /**
     * 测试指定的对象是否是此哈希表中的键。
     *
     * @param   key   possible key
     * @return  <code>true</code> if and only if the specified object
     *          is a key in this hashtable, as determined by the
     *          <tt>equals</tt> method; <code>false</code> otherwise.
     * @throws  NullPointerException  if the key is <code>null</code>
     * @see     #contains(Object)
     */
    public synchronized boolean containsKey(Object key) {
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                return true;
            }
        }
        return false;
    }

    /**
     * 返回指定键映射到的值，如果此映射不包含键的映射，则返回{@code null}。
     *
     * 更正式地说，如果此映射包含从密钥{@code k}到值{@code v}的映射，使得
     * {@code（key.equals（k））}，则此方法返回{@code v}; 否则返回{@code null}。
     * （最多可以有一个这样的映射。）
     *
     * @param key the key whose associated value is to be returned
     * @return the value to which the specified key is mapped, or
     *         {@code null} if this map contains no mapping for the key
     * @throws NullPointerException if the specified key is null
     * @see     #put(Object, Object)
     */
    @SuppressWarnings("unchecked")
    public synchronized V get(Object key) {
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                return (V)e.value;
            }
        }
        return null;
    }

    /**
     * 要分配的最大数组大小。 一些VM在阵列中保留一些标题字。 
     * 尝试分配更大的数组可能会导致OutOfMemoryError：请求的数组大小超过VM限制
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * 增加此哈希表的容量并在内部重新组织，以便更有效地容纳和访问其条目。 
     * 当散列表中的键数超过此散列表的容量和负载因子时，将自动调用此方法。
     */
    @SuppressWarnings("unchecked")
    protected void rehash() {
        int oldCapacity = table.length;
        Entry<?,?>[] oldMap = table;

        // overflow-conscious code
        int newCapacity = (oldCapacity << 1) + 1;
        if (newCapacity - MAX_ARRAY_SIZE > 0) {
            if (oldCapacity == MAX_ARRAY_SIZE)
                // Keep running with MAX_ARRAY_SIZE buckets
                return;
            newCapacity = MAX_ARRAY_SIZE;
        }
        Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];

        modCount++;
        threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
        table = newMap;

        for (int i = oldCapacity ; i-- > 0 ;) {
            for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
                Entry<K,V> e = old;
                old = old.next;

                int index = (e.hash & 0x7FFFFFFF) % newCapacity;
                e.next = (Entry<K,V>)newMap[index];
                newMap[index] = e;
            }
        }
    }

    private void addEntry(int hash, K key, V value, int index) {
        modCount++;

        Entry<?,?> tab[] = table;
        if (count >= threshold) {
            // Rehash the table if the threshold is exceeded
            rehash();

            tab = table;
            hash = key.hashCode();
            index = (hash & 0x7FFFFFFF) % tab.length;
        }

        // Creates the new entry.
        @SuppressWarnings("unchecked")
        Entry<K,V> e = (Entry<K,V>) tab[index];
        tab[index] = new Entry<>(hash, key, value, e);
        count++;
    }

    /**
     * 将指定的键映射到此哈希表中的指定值。 密钥和值都不能为空。
     *
     * 可以通过使用等于原始键的键调用get方法来检索该值。
     *
     * @param      key     the hashtable key
     * @param      value   the value
     * @return     the previous value of the specified key in this hashtable,
     *             or <code>null</code> if it did not have one
     * @exception  NullPointerException  if the key or value is
     *               <code>null</code>
     * @see     Object#equals(Object)
     * @see     #get(Object)
     */
    public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
    }

    /**
     * 从此哈希表中删除键（及其对应的值）。 如果密钥不在哈希表中，则此方法不执行任何操作。
     *
     * @param   key   the key that needs to be removed
     * @return  the value to which the key had been mapped in this hashtable,
     *          or <code>null</code> if the key did not have a mapping
     * @throws  NullPointerException  if the key is <code>null</code>
     */
    public synchronized V remove(Object key) {
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> e = (Entry<K,V>)tab[index];
        for(Entry<K,V> prev = null ; e != null ; prev = e, e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                modCount++;
                if (prev != null) {
                    prev.next = e.next;
                } else {
                    tab[index] = e.next;
                }
                count--;
                V oldValue = e.value;
                e.value = null;
                return oldValue;
            }
        }
        return null;
    }

    /**
     * 将指定映射中的所有映射复制到此哈希表。 
     * 这些映射将替换此哈希表对当前在指定映射中的任何键所具有的任何映射。
     *
     * @param t mappings to be stored in this map
     * @throws NullPointerException if the specified map is null
     * @since 1.2
     */
    public synchronized void putAll(Map<? extends K, ? extends V> t) {
        for (Map.Entry<? extends K, ? extends V> e : t.entrySet())
            put(e.getKey(), e.getValue());
    }

    /**
     * Clears this hashtable so that it contains no keys.
     */
    public synchronized void clear() {
        Entry<?,?> tab[] = table;
        modCount++;
        for (int index = tab.length; --index >= 0; )
            tab[index] = null;
        count = 0;
    }

    /**
     * 创建此哈希表的浅表副本。 复制哈希表本身的所有结构，但不克隆键和值。 
     * 这是一项相对昂贵的操作。
     *
     * @return  a clone of the hashtable
     */
    public synchronized Object clone() {
        try {
            Hashtable<?,?> t = (Hashtable<?,?>)super.clone();
            t.table = new Entry<?,?>[table.length];
            for (int i = table.length ; i-- > 0 ; ) {
                t.table[i] = (table[i] != null)
                    ? (Entry<?,?>) table[i].clone() : null;
            }
            t.keySet = null;
            t.entrySet = null;
            t.values = null;
            t.modCount = 0;
            return t;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError(e);
        }
    }

    /**
     * 以一组条目的形式返回此Hashtable对象的字符串表示形式，用大括号括起并用
     * ASCII字符“ ”（逗号和空格）分隔。 每个条目都呈现为键，等号=和关联元素，其
     * 中toString方法用于将键和元素转换为字符串。
     *
     * @return  a string representation of this hashtable
     */
    public synchronized String toString() {
        int max = size() - 1;
        if (max == -1)
            return "{}";

        StringBuilder sb = new StringBuilder();
        Iterator<Map.Entry<K,V>> it = entrySet().iterator();

        sb.append('{');
        for (int i = 0; ; i++) {
            Map.Entry<K,V> e = it.next();
            K key = e.getKey();
            V value = e.getValue();
            sb.append(key   == this ? "(this Map)" : key.toString());
            sb.append('=');
            sb.append(value == this ? "(this Map)" : value.toString());

            if (i == max)
                return sb.append('}').toString();
            sb.append(", ");
        }
    }


    private <T> Enumeration<T> getEnumeration(int type) {
        if (count == 0) {
            return Collections.emptyEnumeration();
        } else {
            return new Enumerator<>(type, false);
        }
    }

    private <T> Iterator<T> getIterator(int type) {
        if (count == 0) {
            return Collections.emptyIterator();
        } else {
            return new Enumerator<>(type, true);
        }
    }

    // Views

    /**
     * 初始化每个字段以在第一次请求此视图时包含相应视图的实例。 
     * 视图是无状态的，因此没有理由创建多个视图。
     */
    private transient volatile Set<K> keySet;
    private transient volatile Set<Map.Entry<K,V>> entrySet;
    private transient volatile Collection<V> values;

    /**
     * 返回此映射中包含的键的{@link Set}视图。 
     * 该集由地图支持，因此对地图的更改将反映在集中，反之亦然。 如果在对集
     * 合进行迭代时修改了映射（除了通过迭代器自己的remove操作），迭代的结果
     * 是未定义的。 该集支持元素删除，它通过Iterator.remove，Set.*remove，removeAll，
     * retainAll和clear操作从地图中删除相应的映射。 
     * 它不支持add或addAll perations。
     *
     * @since 1.2
     */
    public Set<K> keySet() {
        if (keySet == null)
            keySet = Collections.synchronizedSet(new KeySet(), this);
        return keySet;
    }

    private class KeySet extends AbstractSet<K> {
        public Iterator<K> iterator() {
            return getIterator(KEYS);
        }
        public int size() {
            return count;
        }
        public boolean contains(Object o) {
            return containsKey(o);
        }
        public boolean remove(Object o) {
            return Hashtable.this.remove(o) != null;
        }
        public void clear() {
            Hashtable.this.clear();
        }
    }

    /**
     * 返回此映射中包含的映射的{@link Set}视图。 
     * 该集由地图支持，因此对地图的更改将反映在集中，反之亦然。 如果在对集
     * 合进行迭代时修改了映射（除非通过迭代器自己的remove操作，或者通过迭
     * 代器返回的映射条目上的setValue操作），迭代的结果是未定义的。 
     * 该集支持元素删除，它通过Iterator.remove，Set.remove，removeAll，retainAll
     * 和clear操作从地图中删除相应的映射。 它不支持add或addAll操作。
     *
     * @since 1.2
     */
    public Set<Map.Entry<K,V>> entrySet() {
        if (entrySet==null)
            entrySet = Collections.synchronizedSet(new EntrySet(), this);
        return entrySet;
    }

    private class EntrySet extends AbstractSet<Map.Entry<K,V>> {
        public Iterator<Map.Entry<K,V>> iterator() {
            return getIterator(ENTRIES);
        }

        public boolean add(Map.Entry<K,V> o) {
            return super.add(o);
        }

        public boolean contains(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> entry = (Map.Entry<?,?>)o;
            Object key = entry.getKey();
            Entry<?,?>[] tab = table;
            int hash = key.hashCode();
            int index = (hash & 0x7FFFFFFF) % tab.length;

            for (Entry<?,?> e = tab[index]; e != null; e = e.next)
                if (e.hash==hash && e.equals(entry))
                    return true;
            return false;
        }

        public boolean remove(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> entry = (Map.Entry<?,?>) o;
            Object key = entry.getKey();
            Entry<?,?>[] tab = table;
            int hash = key.hashCode();
            int index = (hash & 0x7FFFFFFF) % tab.length;

            @SuppressWarnings("unchecked")
            Entry<K,V> e = (Entry<K,V>)tab[index];
            for(Entry<K,V> prev = null; e != null; prev = e, e = e.next) {
                if (e.hash==hash && e.equals(entry)) {
                    modCount++;
                    if (prev != null)
                        prev.next = e.next;
                    else
                        tab[index] = e.next;

                    count--;
                    e.value = null;
                    return true;
                }
            }
            return false;
        }

        public int size() {
            return count;
        }

        public void clear() {
            Hashtable.this.clear();
        }
    }

    /**
     * 返回此映射中包含的值的{@link Collection}视图。 
     * 该集合由地图支持，因此对地图的更改将反映在集合中，反之亦然。 如果在
     * 对集合进行迭代时修改了映射（除了通过迭代器自己的remove操作），迭代
     * 的结果是未定义的。 该集合支持元素删除，它通过
     * Iterator.remove，Collection.remove，removeAll，retainAll和clear
     * 操作从地图中删除相应的映射。 它不支持add或addAll操作。
     *
     * @since 1.2
     */
    public Collection<V> values() {
        if (values==null)
            values = Collections.synchronizedCollection(new ValueCollection(),
                                                        this);
        return values;
    }

    private class ValueCollection extends AbstractCollection<V> {
        public Iterator<V> iterator() {
            return getIterator(VALUES);
        }
        public int size() {
            return count;
        }
        public boolean contains(Object o) {
            return containsValue(o);
        }
        public void clear() {
            Hashtable.this.clear();
        }
    }

    // Comparison and hashing

    /**
     * 根据Map接口中的定义，将指定的Object与此Map进行相等性比较。
     *
     * @param  o object to be compared for equality with this hashtable
     * @return true if the specified Object is equal to this Map
     * @see Map#equals(Object)
     * @since 1.2
     */
    public synchronized boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Map))
            return false;
        Map<?,?> t = (Map<?,?>) o;
        if (t.size() != size())
            return false;

        try {
            Iterator<Map.Entry<K,V>> i = entrySet().iterator();
            while (i.hasNext()) {
                Map.Entry<K,V> e = i.next();
                K key = e.getKey();
                V value = e.getValue();
                if (value == null) {
                    if (!(t.get(key)==null && t.containsKey(key)))
                        return false;
                } else {
                    if (!value.equals(t.get(key)))
                        return false;
                }
            }
        } catch (ClassCastException unused)   {
            return false;
        } catch (NullPointerException unused) {
            return false;
        }

        return true;
    }

    /**
     * 根据Map接口中的定义返回此Map的哈希码值。
     *
     * @see Map#hashCode()
     * @since 1.2
     */
    public synchronized int hashCode() {
        /*
         * 此代码检测由计算自引用哈希表的哈希代码引起的递归，并防止否则
         * 将导致的堆栈溢出。 这允许某些具有自引用哈希表的1.1版applet工作。 
         * 此代码滥用loadFactor字段作为hashCode in progress标志执行双重任务，
         * 以免恶化空间性能。 负负载因子表示哈希码计算正在进行中。
         */
        int h = 0;
        if (count == 0 || loadFactor < 0)
            return h;  // Returns zero

        loadFactor = -loadFactor;  // Mark hashCode computation in progress
        Entry<?,?>[] tab = table;
        for (Entry<?,?> entry : tab) {
            while (entry != null) {
                h += entry.hashCode();
                entry = entry.next;
            }
        }

        loadFactor = -loadFactor;  // Mark hashCode computation complete

        return h;
    }

    @Override
    public synchronized V getOrDefault(Object key, V defaultValue) {
        V result = get(key);
        return (null == result) ? defaultValue : result;
    }

    @SuppressWarnings("unchecked")
    @Override
    public synchronized void forEach(BiConsumer<? super K, ? super V> action) {
        Objects.requireNonNull(action);     // explicit check required in case
                                            // table is empty.
        final int expectedModCount = modCount;

        Entry<?, ?>[] tab = table;
        for (Entry<?, ?> entry : tab) {
            while (entry != null) {
                action.accept((K)entry.key, (V)entry.value);
                entry = entry.next;

                if (expectedModCount != modCount) {
                    throw new ConcurrentModificationException();
                }
            }
        }
    }

    @SuppressWarnings("unchecked")
    @Override
    public synchronized void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
        Objects.requireNonNull(function);     // explicit check required in case
                                              // table is empty.
        final int expectedModCount = modCount;

        Entry<K, V>[] tab = (Entry<K, V>[])table;
        for (Entry<K, V> entry : tab) {
            while (entry != null) {
                entry.value = Objects.requireNonNull(
                    function.apply(entry.key, entry.value));
                entry = entry.next;

                if (expectedModCount != modCount) {
                    throw new ConcurrentModificationException();
                }
            }
        }
    }

    @Override
    public synchronized V putIfAbsent(K key, V value) {
        Objects.requireNonNull(value);

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        for (; entry != null; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                if (old == null) {
                    entry.value = value;
                }
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
    }

    @Override
    public synchronized boolean remove(Object key, Object value) {
        Objects.requireNonNull(value);

        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> e = (Entry<K,V>)tab[index];
        for (Entry<K,V> prev = null; e != null; prev = e, e = e.next) {
            if ((e.hash == hash) && e.key.equals(key) && e.value.equals(value)) {
                modCount++;
                if (prev != null) {
                    prev.next = e.next;
                } else {
                    tab[index] = e.next;
                }
                count--;
                e.value = null;
                return true;
            }
        }
        return false;
    }

    @Override
    public synchronized boolean replace(K key, V oldValue, V newValue) {
        Objects.requireNonNull(oldValue);
        Objects.requireNonNull(newValue);
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> e = (Entry<K,V>)tab[index];
        for (; e != null; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                if (e.value.equals(oldValue)) {
                    e.value = newValue;
                    return true;
                } else {
                    return false;
                }
            }
        }
        return false;
    }

    @Override
    public synchronized V replace(K key, V value) {
        Objects.requireNonNull(value);
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> e = (Entry<K,V>)tab[index];
        for (; e != null; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                V oldValue = e.value;
                e.value = value;
                return oldValue;
            }
        }
        return null;
    }

    @Override
    public synchronized V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction) {
        Objects.requireNonNull(mappingFunction);

        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> e = (Entry<K,V>)tab[index];
        for (; e != null; e = e.next) {
            if (e.hash == hash && e.key.equals(key)) {
                // Hashtable not accept null value
                return e.value;
            }
        }

        V newValue = mappingFunction.apply(key);
        if (newValue != null) {
            addEntry(hash, key, newValue, index);
        }

        return newValue;
    }

    @Override
    public synchronized V computeIfPresent(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);

        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> e = (Entry<K,V>)tab[index];
        for (Entry<K,V> prev = null; e != null; prev = e, e = e.next) {
            if (e.hash == hash && e.key.equals(key)) {
                V newValue = remappingFunction.apply(key, e.value);
                if (newValue == null) {
                    modCount++;
                    if (prev != null) {
                        prev.next = e.next;
                    } else {
                        tab[index] = e.next;
                    }
                    count--;
                } else {
                    e.value = newValue;
                }
                return newValue;
            }
        }
        return null;
    }

    @Override
    public synchronized V compute(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);

        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> e = (Entry<K,V>)tab[index];
        for (Entry<K,V> prev = null; e != null; prev = e, e = e.next) {
            if (e.hash == hash && Objects.equals(e.key, key)) {
                V newValue = remappingFunction.apply(key, e.value);
                if (newValue == null) {
                    modCount++;
                    if (prev != null) {
                        prev.next = e.next;
                    } else {
                        tab[index] = e.next;
                    }
                    count--;
                } else {
                    e.value = newValue;
                }
                return newValue;
            }
        }

        V newValue = remappingFunction.apply(key, null);
        if (newValue != null) {
            addEntry(hash, key, newValue, index);
        }

        return newValue;
    }

    @Override
    public synchronized V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);

        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> e = (Entry<K,V>)tab[index];
        for (Entry<K,V> prev = null; e != null; prev = e, e = e.next) {
            if (e.hash == hash && e.key.equals(key)) {
                V newValue = remappingFunction.apply(e.value, value);
                if (newValue == null) {
                    modCount++;
                    if (prev != null) {
                        prev.next = e.next;
                    } else {
                        tab[index] = e.next;
                    }
                    count--;
                } else {
                    e.value = newValue;
                }
                return newValue;
            }
        }

        if (value != null) {
            addEntry(hash, key, value, index);
        }

        return value;
    }

    /**
     * 将Hashtable的状态保存到流中（即序列化）。
     *
     * @serialData The <i>capacity</i> of the Hashtable (the length of the
     *             bucket array) is emitted (int), followed by the
     *             <i>size</i> of the Hashtable (the number of key-value
     *             mappings), followed by the key (Object) and value (Object)
     *             for each key-value mapping represented by the Hashtable
     *             The key-value mappings are emitted in no particular order.
     */
    private void writeObject(java.io.ObjectOutputStream s)
            throws IOException {
        Entry<Object, Object> entryStack = null;

        synchronized (this) {
            // Write out the threshold and loadFactor
            s.defaultWriteObject();

            // Write out the length and count of elements
            s.writeInt(table.length);
            s.writeInt(count);

            // Stack copies of the entries in the table
            for (int index = 0; index < table.length; index++) {
                Entry<?,?> entry = table[index];

                while (entry != null) {
                    entryStack =
                        new Entry<>(0, entry.key, entry.value, entryStack);
                    entry = entry.next;
                }
            }
        }

        // Write out the key/value objects from the stacked entries
        while (entryStack != null) {
            s.writeObject(entryStack.key);
            s.writeObject(entryStack.value);
            entryStack = entryStack.next;
        }
    }

    /**
     * 从流中重构Hashtable（即，反序列化它）。
     */
    private void readObject(java.io.ObjectInputStream s)
         throws IOException, ClassNotFoundException
    {
        // 读入阈值和loadFactor
        s.defaultReadObject();

        // 验证loadFactor（忽略阈值 - 它将被重新计算）
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new StreamCorruptedException("Illegal Load: " + loadFactor);

        // 读取数组的原始长度和元素数量
        int origlength = s.readInt();
        int elements = s.readInt();

        // Validate # of elements
        if (elements < 0)
            throw new StreamCorruptedException("Illegal # of Elements: " + elements);

        // Clamp原始长度要超过elements / loadFactor
        // （这是通过自动增长强制执行的不变量）
        origlength = Math.max(origlength, (int)(elements / loadFactor) + 1);

        // 用一点5％+ 3的空间计算新的长度，但不要大于夹紧的原始长度。
        // 如果长度足够大，则将长度设为奇数，这有助于分配条目。 
        // 防止结束为零的长度，这是无效的。
        int length = (int)((elements + elements / 20) / loadFactor) + 3;
        if (length > elements && (length & 1) == 0)
            length--;
        length = Math.min(length, origlength);

        if (length < 0) { // overflow
            length = origlength;
        }

        // 查Map.Entry [] .class，因为它是我们实际创建的最近的公共类型。
        SharedSecrets.getJavaOISAccess().checkArray(s, Map.Entry[].class, length);
        table = new Entry<?,?>[length];
        threshold = (int)Math.min(length * loadFactor, MAX_ARRAY_SIZE + 1);
        count = 0;

        // Read the number of elements and then all the key/value objects
        for (; elements > 0; elements--) {
            @SuppressWarnings("unchecked")
                K key = (K)s.readObject();
            @SuppressWarnings("unchecked")
                V value = (V)s.readObject();
            // sync is eliminated for performance
            reconstitutionPut(table, key, value);
        }
    }

    /**
     * readObject使用的put方法。 
     * 这是因为put是可覆盖的，不应该在readObject中调用，因为子类尚未初始化。 
     * 这与常规put方法有几点不同。 由于表中最初的元素数量是已知的，因此不需要检
     * 查重新散列。 modCount没有增加，也没有同步，因为我们正在创建一个新实例。 
     * 此外，不需要返回值。
     */
    private void reconstitutionPut(Entry<?,?>[] tab, K key, V value)
        throws StreamCorruptedException
    {
        if (value == null) {
            throw new java.io.StreamCorruptedException();
        }
        // Makes sure the key is not already in the hashtable.
        // This should not happen in deserialized version.
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                throw new java.io.StreamCorruptedException();
            }
        }
        // Creates the new entry.
        @SuppressWarnings("unchecked")
            Entry<K,V> e = (Entry<K,V>)tab[index];
        tab[index] = new Entry<>(hash, key, value, e);
        count++;
    }

    /**
     * 哈希表桶冲突列表条目
     */
    private static class Entry<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Entry<K,V> next;

        protected Entry(int hash, K key, V value, Entry<K,V> next) {
            this.hash = hash;
            this.key =  key;
            this.value = value;
            this.next = next;
        }

        @SuppressWarnings("unchecked")
        protected Object clone() {
            return new Entry<>(hash, key, value,
                                  (next==null ? null : (Entry<K,V>) next.clone()));
        }

        // Map.Entry Ops

        public K getKey() {
            return key;
        }

        public V getValue() {
            return value;
        }

        public V setValue(V value) {
            if (value == null)
                throw new NullPointerException();

            V oldValue = this.value;
            this.value = value;
            return oldValue;
        }

        public boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;

            return (key==null ? e.getKey()==null : key.equals(e.getKey())) &&
               (value==null ? e.getValue()==null : value.equals(e.getValue()));
        }

        public int hashCode() {
            return hash ^ Objects.hashCode(value);
        }

        public String toString() {
            return key.toString()+"="+value.toString();
        }
    }

    // Types of Enumerations/Iterations
    private static final int KEYS = 0;
    private static final int VALUES = 1;
    private static final int ENTRIES = 2;

    /**
     * 哈希表枚举器类。 
     * 此类实现Enumeration和Iterator接口，但可以在禁用Iterator方法的情况下创
     * 建单个实例。 这是必要的，以避免通过传递枚举无意中增加授予用户的功能。
     */
    private class Enumerator<T> implements Enumeration<T>, Iterator<T> {
        Entry<?,?>[] table = Hashtable.this.table;
        int index = table.length;
        Entry<?,?> entry;
        Entry<?,?> lastReturned;
        int type;

        /**
         * 指示此枚举器是用作迭代器还是枚举。 （true  - > Iterator）。
         */
        boolean iterator;

        /**
         * 迭代器认为支持Hashtable应具有的modCount值。
         * 如果违反了此期望，则迭代器已检测到并发修改。
         */
        protected int expectedModCount = modCount;

        Enumerator(int type, boolean iterator) {
            this.type = type;
            this.iterator = iterator;
        }

        public boolean hasMoreElements() {
            Entry<?,?> e = entry;
            int i = index;
            Entry<?,?>[] t = table;
            /* Use locals for faster loop iteration */
            while (e == null && i > 0) {
                e = t[--i];
            }
            entry = e;
            index = i;
            return e != null;
        }

        @SuppressWarnings("unchecked")
        public T nextElement() {
            Entry<?,?> et = entry;
            int i = index;
            Entry<?,?>[] t = table;
            /* Use locals for faster loop iteration */
            while (et == null && i > 0) {
                et = t[--i];
            }
            entry = et;
            index = i;
            if (et != null) {
                Entry<?,?> e = lastReturned = entry;
                entry = e.next;
                return type == KEYS ? (T)e.key : (type == VALUES ? (T)e.value : (T)e);
            }
            throw new NoSuchElementException("Hashtable Enumerator");
        }

        // Iterator methods
        public boolean hasNext() {
            return hasMoreElements();
        }

        public T next() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            return nextElement();
        }

        public void remove() {
            if (!iterator)
                throw new UnsupportedOperationException();
            if (lastReturned == null)
                throw new IllegalStateException("Hashtable Enumerator");
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();

            synchronized(Hashtable.this) {
                Entry<?,?>[] tab = Hashtable.this.table;
                int index = (lastReturned.hash & 0x7FFFFFFF) % tab.length;

                @SuppressWarnings("unchecked")
                Entry<K,V> e = (Entry<K,V>)tab[index];
                for(Entry<K,V> prev = null; e != null; prev = e, e = e.next) {
                    if (e == lastReturned) {
                        modCount++;
                        expectedModCount++;
                        if (prev == null)
                            tab[index] = e.next;
                        else
                            prev.next = e.next;
                        count--;
                        lastReturned = null;
                        return;
                    }
                }
                throw new ConcurrentModificationException();
            }
        }
    }
}

```