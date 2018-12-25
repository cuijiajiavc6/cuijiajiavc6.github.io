```java 
package java.util;

import java.util.function.BiConsumer;
import java.util.function.BiFunction;
import java.util.function.Function;
import java.io.Serializable;

/**
 * 将键映射到值的对象。 地图不能包含重复的键; 每个键最多可以映射一个值。
 *
 * 这个接口取代了Dictionary类，它是一个完全抽象的类而不是接口。
 *
 * Map接口提供三个集合视图，允许将地图的内容视为一组键，值集合或键值映射集。
 * 地图的顺序定义为地图集合视图上的迭代器返回其元素的顺序。 一些地图实现，
 * 比如TreeMap类，对它们的顺序做出了特定的保证; 其他人，比如HashMap类，没有。
 *
 * 注意：如果将可变对象用作映射键，则必须非常小心。 
 * 如果在对象是地图中的键的同时以影响等于比较的方式更改对象的值，则不指定映射的行为。 
 * 这种禁令的一个特例是，地图不允许将自己作为一个关键词。 
 * 虽然允许映射将自身包含为值，但建议极其谨慎：equals和hashCode方法不再在这样的映射
 * 上很好地定义。
 *
 * 所有通用映射实现类都应该提供两个“标准”构造函数：一个void（无参数）构造函数，它
 * 创建一个空映射，一个构造函数具有一个类型为Map的参数，它创建一个具有相同键值的新
 * 映射 映射作为其论点。 实际上，后一个构造函数允许用户复制任何地图，生成所需类的等
 * 效地图。 没有办法强制执行此建议（因为接口不能包含构造函数），但JDK中的所有通用映
 * 射实现都符合。
 *
 *
 * 如果此映射不支持该操作，则此接口中包含的“破坏性”方法（即修改它们操作的映射的方法）
 * 被指定为抛出UnsupportedOperationException。 
 * 如果是这种情况，如果调用对地图没有影响，这些方法可能（但不是必须）抛出
 * UnsupportedOperationException。 例如，如果要映射“映射”的映射为空，则在不可修改的
 * 映射上调用{@link #putAll（Map）}方法可能（但不是必须）抛出异常。
 *
 * 某些地图实现对它们可能包含的键和值有限制。 例如，某些实现禁止空键和值，有些实现对
 * 其键的类型有限制。 尝试插入不合格的键或值会引发未经检查的异常，通常为
 * NullPointerException或ClassCastException。 尝试查询不合格的键或值的存在可能会引发
 * 异常，或者它可能只是返回false; 一些实现将展示前一种行为，一些将展示后者。 更一般
 * 地，尝试对不合格的密钥或值进行操作，其完成不会导致将不合格的元素插入到映射中，可
 * 能会引发异常，或者可能在实现的选择中成功。 此类异常在此接口的规范中标记为“可选”。
 * 
 *
 * Collections Framework接口中的许多方法都是根据{@link Object＃equals（Object）equals}
 * 方法定义的。例如，
 * {@link #containsKey（Object）containsKey（Object key）}方法的规范说：“当且仅当此映
 * 射包含密钥k的映射时才返回true（key == null？k = = null：key.equals（k））。“不应将
 * 此规范解释为暗示使用非空参数键调用Map.containsKey将导致为任何键k调用key.equals（k）。
 * 实现可以自由地实现优化，从而避免等于调用，例如，通过首先比较两个密钥的哈希码。 
 * （{@link Object＃hashCode（）}规范保证具有不等哈希码的两个对象不能相等。）更一般地，
 * 各种Collections Framework接口的实现可以自由地利用底层{@link对象的指定行为。在实现者
 * 认为合适的地方。
 *
 *
 * 执行映射递归遍历的某些映射操作可能会失败，并且映射直接或间接包含自身的自引用实例例
 * 外。 这包括{@code clone（）}，{@ code equals（）}，{@ code hashCode（）}和
 * {@code toString（）}方法。 实现可以可选地处理自引用场景，但是大多数当前实现不这样做。
 *
 *
 * <p>This interface is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 *
 * @author  Josh Bloch
 * @see HashMap
 * @see TreeMap
 * @see Hashtable
 * @see SortedMap
 * @see Collection
 * @see Set
 * @since 1.2
 */
public interface Map<K,V> {
    // Query Operations

    /**
     * 返回此映射中键 - 值映射的数量。 如果映射包含多个Integer.MAX_VALUE元素，则返回
     * <tt>Integer.MAX_VALUE</tt>.
     *
     * @return the number of key-value mappings in this map
     */
    int size();

    /**
     * 如果此映射不包含键 - 值映射，则返回true。
     *
     * @return <tt>true</tt> if this map contains no key-value mappings
     */
    boolean isEmpty();

    /**
     * 如果此映射包含指定键的映射，则返回true。 更正式地，当且仅当此映射
     * 包含键k的映射时才返回true（key == null？k == null：key.equals（k））。
     * （最多可以有一个这样的映射。）
     *
     * @param key key whose presence in this map is to be tested
     * @return <tt>true</tt> if this map contains a mapping for the specified
     *         key
     * @throws ClassCastException if the key is of an inappropriate type for
     *         this map
     * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified key is null and this map
     *         does not permit null keys
     * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     */
    boolean containsKey(Object key);

    /**
     * 如果此映射将一个或多个键映射到指定值，则返回true。 
     * 更正式地，当且仅当此映射包含至少一个到值v的映射时才返回true
     *（value == null？v == null：value.equals（v））。 对于Map接口的大多数实现，
     * 此操作可能需要地图大小的线性时间。
     *
     * @param value value whose presence in this map is to be tested
     * @return <tt>true</tt> if this map maps one or more keys to the
     *         specified value
     * @throws ClassCastException if the value is of an inappropriate type for
     *         this map
     * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified value is null and this
     *         map does not permit null values
     * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     */
    boolean containsValue(Object value);

    /**
     * 返回指定键映射到的值，如果此映射不包含键的映射，则返回{@code null}。
     *
     * 更正式地说，如果此映射包含从密钥{@code k}到值{@code v}的映射，使得
     * {@code（key == null？k == null：key.equals（k））}， 然后这个方法返回{@code v}; 
     * 否则返回{@code null}。 （最多可以有一个这样的映射。）
     *
     * 如果此映射允许空值，则返回值{@code null}不一定表示映射不包含该键的映射; 地图也可
     * 能明确地将密钥映射到{@code null}。 {@link #containsKey containsKey}操作可用于区
     * 分这两种情况。
     *
     * @param key the key whose associated value is to be returned
     * @return the value to which the specified key is mapped, or
     *         {@code null} if this map contains no mapping for the key
     * @throws ClassCastException if the key is of an inappropriate type for
     *         this map
     * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified key is null and this map
     *         does not permit null keys
     * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     */
    V get(Object key);

    // Modification Operations

    /**
     * 将指定的值与此映射中的指定键相关联（可选操作）。 如果映射先前包含键的映射，则旧值将
     * 替换为指定的值。 （当且仅当{@link #containsKey（Object）m.containsKey（k）}返回true时，
     * 才称映射m包含密钥k的映射。）
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>,
     *         if the implementation supports <tt>null</tt> values.)
     * @throws UnsupportedOperationException if the <tt>put</tt> operation
     *         is not supported by this map
     * @throws ClassCastException if the class of the specified key or value
     *         prevents it from being stored in this map
     * @throws NullPointerException if the specified key or value is null
     *         and this map does not permit null keys or values
     * @throws IllegalArgumentException if some property of the specified key
     *         or value prevents it from being stored in this map
     */
    V put(K key, V value);

    /**
     * 如果存在，则从该映射中移除键的映射（可选操作）。 更正式地说，如果此映射包含从键
     * k到值v的映射，使得（key == null？k == null：key.equals（k）），则删除该映射。 
     * （地图最多可以包含一个这样的映射。）
     *
     * 返回此映射先前与该键关联的值，如果映射不包含该键的映射，则返回null。
     *
     *
     * 如果此映射允许空值，则返回值null不一定表示映射不包含键的映射; 地图也可能将键显式
     * 映射为null。
     *
     * 一旦调用返回，映射将不包含指定键的映射。
     *
     * @param key key whose mapping is to be removed from the map
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     * @throws UnsupportedOperationException if the <tt>remove</tt> operation
     *         is not supported by this map
     * @throws ClassCastException if the key is of an inappropriate type for
     *         this map
     * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified key is null and this
     *         map does not permit null keys
     * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     */
    V remove(Object key);


    // Bulk Operations

    /**
     * 将指定映射中的所有映射复制到此映射（可选操作）。 
     * 对于在指定映射中从键k到值v的每个映射，此调用的效果等同于在此映射上调用
     * {@link #put（Object，Object）put（k，v）}的效果。 如果在操作过程中修改
     * 了指定的映射，则此操作的行为是不确定的。
     *
     * @param m mappings to be stored in this map
     * @throws UnsupportedOperationException if the <tt>putAll</tt> operation
     *         is not supported by this map
     * @throws ClassCastException if the class of a key or value in the
     *         specified map prevents it from being stored in this map
     * @throws NullPointerException if the specified map is null, or if
     *         this map does not permit null keys or values, and the
     *         specified map contains null keys or values
     * @throws IllegalArgumentException if some property of a key or value in
     *         the specified map prevents it from being stored in this map
     */
    void putAll(Map<? extends K, ? extends V> m);

    /**
     * 从此映射中删除所有映射（可选操作）。 此调用返回后，映射将为空。
     *
     * @throws UnsupportedOperationException if the <tt>clear</tt> operation
     *         is not supported by this map
     */
    void clear();


    // Views

    /**
     * 返回此映射中包含的键的{@link Set}视图。 该集由地图支持，因此对地图的更改将
     * 反映在集中，反之亦然。 如果在对集合进行迭代时修改了映射（除了通过迭代器自
     * 己的remove操作），迭代的结果是未定义的。 该集支持元素删除，它通过
     * Iterator.remove，Set.remove，removeAll，retainAll和clear操作从地图中删除相
     * 应的映射。 它不支持add或addAll操作。
     *
     * @return a set view of the keys contained in this map
     */
    Set<K> keySet();

    /**
     * 返回此映射中包含的值的{@link Collection}视图。 该集合由地图支持，因此对地图
     * 的更改将反映在集合中，反之亦然。 如果在对集合进行迭代时修改了映射（除了通过
     * 迭代器自己的remove操作），迭代的结果是未定义的。 该集合支持元素删除，它通过
     * Iterator.remove，Collection.remove，removeAll，retainAll和clear操作从地图中
     * 删除相应的映射。 它不支持add或addAll操作。
     *
     * @return a collection view of the values contained in this map
     */
    Collection<V> values();

    /**
     * 返回此映射中包含的映射的{@link Set}视图。 该集由地图支持，因此对地图的更改将
     * 反映在集中，反之亦然。 如果在对集合进行迭代时修改了映射（除非通过迭代器自己
     * 的remove操作，或者通过迭代器返回的映射条目上的setValue操作），迭代的结果是未
     * 定义的。 该集支持元素删除，它通过Iterator.remove，Set.remove，removeAll，
     * retainAll和clear操作从地图中删除相应的映射。 它不支持add或addAll操作。
     *
     * @return a set view of the mappings contained in this map
     */
    Set<Map.Entry<K, V>> entrySet();

    /**
     * 映射条目（键值对）。 Map.entrySet方法返回地图的集合视图，其元素属于此类。 
     * 获取对映射条目的引用的唯一方法是来自此collection-view的迭代器。 这些Map.Entry
     * 对象仅在迭代期间有效; 更正式地说，如果在迭代器返回条目后修改了支持映射，则映射
     * 条目的行为是未定义的，除非通过映射条目上的setValue操作。
     *
     * @see Map#entrySet()
     * @since 1.2
     */
    interface Entry<K,V> {
        /**
         * 返回与此条目对应的键。
         *
         * @return the key corresponding to this entry
         * @throws IllegalStateException implementations may, but are not
         *         required to, throw this exception if the entry has been
         *         removed from the backing map.
         */
        K getKey();

        /**
         * 返回与此条目对应的值。 如果映射已从支持映射中删除（通过迭代器的remove操作），
         * 则此调用的结果未定义。
         *
         * @return the value corresponding to this entry
         * @throws IllegalStateException implementations may, but are not
         *         required to, throw this exception if the entry has been
         *         removed from the backing map.
         */
        V getValue();

        /**
         * 用指定的值替换此条目对应的值（可选操作）。
         * （写入映射。）如果映射已从映射中删除（通过迭代器的remove操作），则此调用的行
         * 为是未定义的。
         *
         * @param value new value to be stored in this entry
         * @return old value corresponding to the entry
         * @throws UnsupportedOperationException if the <tt>put</tt> operation
         *         is not supported by the backing map
         * @throws ClassCastException if the class of the specified value
         *         prevents it from being stored in the backing map
         * @throws NullPointerException if the backing map does not permit
         *         null values, and the specified value is null
         * @throws IllegalArgumentException if some property of this value
         *         prevents it from being stored in the backing map
         * @throws IllegalStateException implementations may, but are not
         *         required to, throw this exception if the entry has been
         *         removed from the backing map.
         */
        V setValue(V value);

        /**
         * 将指定对象与此条目进行比较以获得相等性。 如果给定对象也是映射条目并且两个
         * 条目表示相同的映射，则返回true。 更正式地，两个条目e1和e2表示相同的映射
         * if<pre>
         *     (e1.getKey()==null ?
         *      e2.getKey()==null : e1.getKey().equals(e2.getKey()))  &amp;&amp;
         *     (e1.getValue()==null ?
         *      e2.getValue()==null : e1.getValue().equals(e2.getValue()))
         * </pre>
         * 这可确保equals方法在Map.Entry接口的不同实现中正常工作。
         *
         * @param o object to be compared for equality with this map entry
         * @return <tt>true</tt> if the specified object is equal to this map
         *         entry
         */
        boolean equals(Object o);

        /**
         * 返回此映射条目的哈希码值。 映射条目e的哈希码被定义为：
         *     (e.getKey()==null   ? 0 : e.getKey().hashCode()) ^
         *     (e.getValue()==null ? 0 : e.getValue().hashCode())
         * </pre>
         * 这确保了e1.equals（e2）意味着对于任何两个条目e1和e2的
         * e1.hashCode（）== e2.hashCode（），如Object.hashCode的常规协定所要求的那样。
         *
         * @return the hash code value for this map entry
         * @see Object#hashCode()
         * @see Object#equals(Object)
         * @see #equals(Object)
         */
        int hashCode();

        /**
         * 返回一个比较器，按键在自然顺序中比较{@link Map.Entry}。
         *
         * 返回的比较器是可序列化的，并在将条目与空键进行比较时抛出{@link NullPointerException}。
         *
         * @param  <K> the {@link Comparable} type of then map keys
         * @param  <V> the type of the map values
         * @return a comparator that compares {@link Map.Entry} in natural order on key.
         * @see Comparable
         * @since 1.8
         */
        public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() {
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> c1.getKey().compareTo(c2.getKey());
        }

        /**
         * 返回一个比较器，按值自然顺序比较{@link Map.Entry}。
         *
         * 返回的比较器是可序列化的，并在比较具有空值的条目时抛出{@link NullPointerException}。
         *
         * @param <K> the type of the map keys
         * @param <V> the {@link Comparable} type of the map values
         * @return a comparator that compares {@link Map.Entry} in natural order on value.
         * @see Comparable
         * @since 1.8
         */
        public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K,V>> comparingByValue() {
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> c1.getValue().compareTo(c2.getValue());
        }

        /**
         * 返回一个比较器，使用给定的{@link Comparator}按键比较{@link Map.Entry}。
         *
         * 如果指定的比较器也是可序列化的，则返回的比较器是可序列化的。
         *
         * @param  <K> the type of the map keys
         * @param  <V> the type of the map values
         * @param  cmp the key {@link Comparator}
         * @return a comparator that compares {@link Map.Entry} by the key.
         * @since 1.8
         */
        public static <K, V> Comparator<Map.Entry<K, V>> comparingByKey(Comparator<? super K> cmp) {
            Objects.requireNonNull(cmp);
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> cmp.compare(c1.getKey(), c2.getKey());
        }

        /**
         * 返回一个比较器，使用给定的{@link Comparator}按值比较{@link Map.Entry}。
         *
         * 如果指定的比较器也是可序列化的，则返回的比较器是可序列化的。
         *
         * @param  <K> the type of the map keys
         * @param  <V> the type of the map values
         * @param  cmp the value {@link Comparator}
         * @return a comparator that compares {@link Map.Entry} by the value.
         * @since 1.8
         */
        public static <K, V> Comparator<Map.Entry<K, V>> comparingByValue(Comparator<? super V> cmp) {
            Objects.requireNonNull(cmp);
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> cmp.compare(c1.getValue(), c2.getValue());
        }
    }

    // Comparison and hashing

    /**
     * 将指定对象与此映射进行比较以获得相等性。 如果给定对象也是一个映射，并且两个映
     * 射表示相同的映射，则返回true。 更正式地说，如果m1.entrySet（）。equals（m2.entrySet（）），
     * 则两个映射m1和m2表示相同的映射。 这确保了equals方法在Map接口的不同实现中正常工作。
     *
     * @param o object to be compared for equality with this map
     * @return <tt>true</tt> if the specified object is equal to this map
     */
    boolean equals(Object o);

    /**
     * 返回此映射的哈希码值。 映射的哈希码被定义为映射的entrySet（）视图中每个条目的
     * 哈希码的总和。 这确保了m1.equals（m2）意味着对于任何两个映射m1和m2的
     * m1.hashCode（）== m2.hashCode（），正如{@link Object＃hashCode}的一般契约所要求
     * 的那样。
     *
     * @return the hash code value for this map
     * @see Map.Entry#hashCode()
     * @see Object#equals(Object)
     * @see #equals(Object)
     */
    int hashCode();

    // Defaultable methods

    /**
     * 返回指定键映射到的值，如果此映射不包含键的映射，则返回{@code defaultValue}。
     *
     * @implSpec
     * 默认实现不保证此方法的同步或原子性属性。 提供原子性保证的任何实现都必须覆盖此
     * 方法并记录其并发属性。
     *
     * @param key the key whose associated value is to be returned
     * @param defaultValue the default mapping of the key
     * @return the value to which the specified key is mapped, or
     * {@code defaultValue} if this map contains no mapping for the key
     * @throws ClassCastException if the key is of an inappropriate type for
     * this map
     * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified key is null and this map
     * does not permit null keys
     * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @since 1.8
     */
    default V getOrDefault(Object key, V defaultValue) {
        V v;
        return (((v = get(key)) != null) || containsKey(key))
            ? v
            : defaultValue;
    }

    /**
     * 对此映射中的每个条目执行给定操作，直到处理完所有条目或操作引发异常。 
     * 除非实现类另有指定，否则将按入口集迭代的顺序执行操作（如果指定了迭代顺序。）
     * 操作抛出的异常将中继到调用方。
     *
     * @implSpec
     * 对于此{@code map}，默认实现等效于：
     * <pre> {@code
     * for (Map.Entry<K, V> entry : map.entrySet())
     *     action.accept(entry.getKey(), entry.getValue());
     * }</pre>
     *
     * 默认实现不保证此方法的同步或原子性属性。 提供原子性保证的任何实现都必须覆盖此方
     * 法并记录其并发属性。
     *
     * @param action The action to be performed for each entry
     * @throws NullPointerException if the specified action is null
     * @throws ConcurrentModificationException if an entry is found to be
     * removed during iteration
     * @since 1.8
     */
    default void forEach(BiConsumer<? super K, ? super V> action) {
        Objects.requireNonNull(action);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
            action.accept(k, v);
        }
    }

    /**
     * 将每个条目的值替换为在该条目上调用给定函数的结果，直到所有条目都已处
     * 理或函数抛出异常。 函数抛出的异常被转发给调用者。
     *
     * @implSpec
     * <p>The default implementation is equivalent to, for this {@code map}:
     * <pre> {@code
     * for (Map.Entry<K, V> entry : map.entrySet())
     *     entry.setValue(function.apply(entry.getKey(), entry.getValue()));
     * }</pre>
     *
     * 默认实现不保证此方法的同步或原子性属性。 提供原子性保证的任何实现都必须覆盖此方法
     * 并记录其并发属性。
     *
     * @param function the function to apply to each entry
     * @throws UnsupportedOperationException if the {@code set} operation
     * is not supported by this map's entry set iterator.
     * @throws ClassCastException if the class of a replacement value
     * prevents it from being stored in this map
     * @throws NullPointerException if the specified function is null, or the
     * specified replacement value is null, and this map does not permit null
     * values
     * @throws ClassCastException if a replacement value is of an inappropriate
     *         type for this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if function or a replacement value is null,
     *         and this map does not permit null keys or values
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws IllegalArgumentException if some property of a replacement value
     *         prevents it from being stored in this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws ConcurrentModificationException if an entry is found to be
     * removed during iteration
     * @since 1.8
     */
    default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
        Objects.requireNonNull(function);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }

            // ise thrown from function is not a cme.
            v = function.apply(k, v);

            try {
                entry.setValue(v);
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
        }
    }

    /**
     * 如果指定的键尚未与值关联（或映射到{@code null}），则将其与给定值关联并返回
     * {@code null}，否则返回当前值。
     *
     * @implSpec
     * 对于此{@code map}，默认实现等效于：
     *
     * <pre> {@code
     * V v = map.get(key);
     * if (v == null)
     *     v = map.put(key, value);
     *
     * return v;
     * }</pre>
     *
     * 默认实现不保证此方法的同步或原子性属性。 提供原子性保证的任何实现都必须覆盖
     * 此方法并记录其并发属性。
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with the specified key, or
     *         {@code null} if there was no mapping for the key.
     *         (A {@code null} return can also indicate that the map
     *         previously associated {@code null} with the key,
     *         if the implementation supports null values.)
     * @throws UnsupportedOperationException if the {@code put} operation
     *         is not supported by this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws ClassCastException if the key or value is of an inappropriate
     *         type for this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified key or value is null,
     *         and this map does not permit null keys or values
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws IllegalArgumentException if some property of the specified key
     *         or value prevents it from being stored in this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @since 1.8
     */
    default V putIfAbsent(K key, V value) {
        V v = get(key);
        if (v == null) {
            v = put(key, value);
        }

        return v;
    }

    /**
     * 仅当指定键当前映射到指定值时才删除该条目的条目。
     *
     * @implSpec
     * 对于此{@code map}，默认实现等效于：
     *
     * <pre> {@code
     * if (map.containsKey(key) && Objects.equals(map.get(key), value)) {
     *     map.remove(key);
     *     return true;
     * } else
     *     return false;
     * }</pre>
     *
     * 默认实现不保证此方法的同步或原子性属性。 提供原子性保证的任何实现都必须覆盖此方
     * 法并记录其并发属性。
     *
     * @param key key with which the specified value is associated
     * @param value value expected to be associated with the specified key
     * @return {@code true} if the value was removed
     * @throws UnsupportedOperationException if the {@code remove} operation
     *         is not supported by this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws ClassCastException if the key or value is of an inappropriate
     *         type for this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified key or value is null,
     *         and this map does not permit null keys or values
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @since 1.8
     */
    default boolean remove(Object key, Object value) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, value) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        remove(key);
        return true;
    }

    /**
     * 仅当前映射到指定值时，才替换指定键的条目。
     *
     * @implSpec
     * 对于此{@code map}，默认实现等效于：
     *
     * <pre> {@code
     * if (map.containsKey(key) && Objects.equals(map.get(key), value)) {
     *     map.put(key, newValue);
     *     return true;
     * } else
     *     return false;
     * }</pre>
     *
     * 如果oldValue为null，则默认实现不会为不支持空值的映射抛出NullPointerException，
     * 除非newValue也为null。
     *
     * 默认实现不保证此方法的同步或原子性属性。 提供原子性保证的任何实现都必须覆盖此方法
     * 并记录其并发属性。
     *
     * @param key key with which the specified value is associated
     * @param oldValue value expected to be associated with the specified key
     * @param newValue value to be associated with the specified key
     * @return {@code true} if the value was replaced
     * @throws UnsupportedOperationException if the {@code put} operation
     *         is not supported by this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws ClassCastException if the class of a specified key or value
     *         prevents it from being stored in this map
     * @throws NullPointerException if a specified key or newValue is null,
     *         and this map does not permit null keys or values
     * @throws NullPointerException if oldValue is null and this map does not
     *         permit null values
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws IllegalArgumentException if some property of a specified key
     *         or value prevents it from being stored in this map
     * @since 1.8
     */
    default boolean replace(K key, V oldValue, V newValue) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, oldValue) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        put(key, newValue);
        return true;
    }

    /**
     * 仅当指定键当前映射到某个值时，才替换该条目的条目。
     *
     * @implSpec
     * 对于此{@code map}，默认实现等效于：
     *
     * <pre> {@code
     * if (map.containsKey(key)) {
     *     return map.put(key, value);
     * } else
     *     return null;
     * }</pre>
     *
     * 默认实现不保证此方法的同步或原子性属性。 提供原子性保证的任何实现都必须覆
     * 盖此方法并记录其并发属性。
     *
     * @param key key with which the specified value is associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with the specified key, or
     *         {@code null} if there was no mapping for the key.
     *         (A {@code null} return can also indicate that the map
     *         previously associated {@code null} with the key,
     *         if the implementation supports null values.)
     * @throws UnsupportedOperationException if the {@code put} operation
     *         is not supported by this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws ClassCastException if the class of the specified key or value
     *         prevents it from being stored in this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified key or value is null,
     *         and this map does not permit null keys or values
     * @throws IllegalArgumentException if some property of the specified key
     *         or value prevents it from being stored in this map
     * @since 1.8
     */
    default V replace(K key, V value) {
        V curValue;
        if (((curValue = get(key)) != null) || containsKey(key)) {
            curValue = put(key, value);
        }
        return curValue;
    }

    /**
     * 如果指定的键尚未与值关联（或映射到{@code null}），则尝试使用给定的映射函数计
     * 算其值，并将其输入此映射，除非{@code null}。
     *
     * 如果函数返回{@code null}，则不记录映射。 如果函数本身抛出（未经检查的）异常，
     * 则重新抛出异常，并且不记录映射。 最常见的用法是构造一个新对象，用作初始映射
     * 值或记忆结果，如：
     *
     * <pre> {@code
     * map.computeIfAbsent(key, k -> new Value(f(k)));
     * }</pre>
     *
     * 或者实现多值映射{@code Map <K，Collection <V >>}，支持每个键的多个值：
     *
     * <pre> {@code
     * map.computeIfAbsent(key, k -> new HashSet<V>()).add(v);
     * }</pre>
     *
     *
     * @implSpec
     * 默认实现等效于此{@code map}的以下步骤，然后返回当前值或{@code null}如果现在不存在：
     *
     * <pre> {@code
     * if (map.get(key) == null) {
     *     V newValue = mappingFunction.apply(key);
     *     if (newValue != null)
     *         map.put(key, newValue);
     * }
     * }</pre>
     *
     * 默认实现不保证此方法的同步或原子性属性。 提供原子性保证的任何实现都必须覆盖此方法
     * 并记录其并发属性。 特别是，子接口{@link java.util.concurrent.ConcurrentMap}
     * 的所有实现都必须记录该函数是否仅在该值不存在时以原子方式应用。
     *
     * @param key key with which the specified value is to be associated
     * @param mappingFunction the function to compute a value
     * @return the current (existing or computed) value associated with
     *         the specified key, or null if the computed value is null
     * @throws NullPointerException if the specified key is null and
     *         this map does not support null keys, or the mappingFunction
     *         is null
     * @throws UnsupportedOperationException if the {@code put} operation
     *         is not supported by this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws ClassCastException if the class of the specified key or value
     *         prevents it from being stored in this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @since 1.8
     */
    default V computeIfAbsent(K key,
            Function<? super K, ? extends V> mappingFunction) {
        Objects.requireNonNull(mappingFunction);
        V v;
        if ((v = get(key)) == null) {
            V newValue;
            if ((newValue = mappingFunction.apply(key)) != null) {
                put(key, newValue);
                return newValue;
            }
        }

        return v;
    }

    /**
     * 如果指定键的值存在且为非null，则尝试在给定键及其当前映射值的情况下计算新映射。
     *
     * 如果函数返回{@code null}，则删除映射。 如果函数本身抛出（未经检查的）异常，则
     * 重新抛出异常，并保持当前映射不变。
     *
     * @implSpec
     * 默认实现等效于为此{@code map}执行以下步骤，然后返回当前值或{@code null}（如果
     * 现在不存在）：
     *
     * <pre> {@code
     * if (map.get(key) != null) {
     *     V oldValue = map.get(key);
     *     V newValue = remappingFunction.apply(key, oldValue);
     *     if (newValue != null)
     *         map.put(key, newValue);
     *     else
     *         map.remove(key);
     * }
     * }</pre>
     *
     * 默认实现不保证此方法的同步或原子性属性。 提供原子性保证的任何实现都必须覆盖此
     * 方法并记录其并发属性。 特别是，子接口
     * {@link java.util.concurrent.ConcurrentMap}的所有实现都必须记录该函数是否仅在
     * 该值不存在时以原子方式应用。
     *
     * @param key key with which the specified value is to be associated
     * @param remappingFunction the function to compute a value
     * @return the new value associated with the specified key, or null if none
     * @throws NullPointerException if the specified key is null and
     *         this map does not support null keys, or the
     *         remappingFunction is null
     * @throws UnsupportedOperationException if the {@code put} operation
     *         is not supported by this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws ClassCastException if the class of the specified key or value
     *         prevents it from being stored in this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @since 1.8
     */
    default V computeIfPresent(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        V oldValue;
        if ((oldValue = get(key)) != null) {
            V newValue = remappingFunction.apply(key, oldValue);
            if (newValue != null) {
                put(key, newValue);
                return newValue;
            } else {
                remove(key);
                return null;
            }
        } else {
            return null;
        }
    }

    /**
     * 尝试计算指定键及其当前映射值的映射（如果没有当前映射，则为{@code null}）。 例如，
     * 要创建或附加{@code String} msg到值映射：
     *
     * <pre> {@code
     * map.compute(key, (k, v) -> (v == null) ? msg : v.concat(msg))}</pre>
     * (Method {@link #merge merge()} is often simpler to use for such purposes.)
     *
     * 如果函数返回{@code null}，则删除映射（或者如果最初不存在则保持不存在）。 
     * 如果函数本身抛出（未经检查的）异常，则重新抛出异常，并保持当前映射不变。
     *
     * @implSpec
     * 默认实现等同于为此{@code map}执行以下步骤，然后返回当前值或{@code null}如果不存在：
     *
     * <pre> {@code
     * V oldValue = map.get(key);
     * V newValue = remappingFunction.apply(key, oldValue);
     * if (oldValue != null ) {
     *    if (newValue != null)
     *       map.put(key, newValue);
     *    else
     *       map.remove(key);
     * } else {
     *    if (newValue != null)
     *       map.put(key, newValue);
     *    else
     *       return null;
     * }
     * }</pre>
     *
     * 默认实现不保证此方法的同步或原子性属性。 提供原子性保证的任何实现都必须覆盖此
     * 方法并记录其并发属性。 特别是，子接口
     * {@link java.util.concurrent.ConcurrentMap}的所有实现都必须记录该函数是否仅在
     * 该值不存在时以原子方式应用。
     *
     * @param key key with which the specified value is to be associated
     * @param remappingFunction the function to compute a value
     * @return the new value associated with the specified key, or null if none
     * @throws NullPointerException if the specified key is null and
     *         this map does not support null keys, or the
     *         remappingFunction is null
     * @throws UnsupportedOperationException if the {@code put} operation
     *         is not supported by this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws ClassCastException if the class of the specified key or value
     *         prevents it from being stored in this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @since 1.8
     */
    default V compute(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        V oldValue = get(key);

        V newValue = remappingFunction.apply(key, oldValue);
        if (newValue == null) {
            // delete mapping
            if (oldValue != null || containsKey(key)) {
                // something to remove
                remove(key);
                return null;
            } else {
                // nothing to do. Leave things as they were.
                return null;
            }
        } else {
            // add or replace old mapping
            put(key, newValue);
            return newValue;
        }
    }

    /**
     * 如果指定的键尚未与值关联或与null关联，则将其与给定的非空值关联。 
     * 否则，将相关值替换为给定重映射函数的结果，或者如果结果为{@code null}则删除。
     * 当组合密钥的多个映射值时，该方法可以是有用的。 例如，要创建或附加
     * {@code String msg}到值映射：
     *
     * <pre> {@code
     * map.merge(key, msg, String::concat)
     * }</pre>
     *
     * 如果函数返回{@code null}，则删除映射。 如果函数本身抛出（未经检查的）
     * 异常，则重新抛出异常，并保持当前映射不变。
     *
     * @implSpec
     * 默认实现等同于为此{@code map}执行以下步骤，然后返回当前值或{@code null}如果不存在：
     *
     * <pre> {@code
     * V oldValue = map.get(key);
     * V newValue = (oldValue == null) ? value :
     *              remappingFunction.apply(oldValue, value);
     * if (newValue == null)
     *     map.remove(key);
     * else
     *     map.put(key, newValue);
     * }</pre>
     *
     * 默认实现不保证此方法的同步或原子性属性。 提供原子性保证的任何实现都必须覆盖此
     * 方法并记录其并发属性。 特别是，子接口
     * {@link java.util.concurrent.ConcurrentMap}的所有实现都必须记录该函数是否仅在
     * 该值不存在时以原子方式应用。
     *
     * @param key key with which the resulting value is to be associated
     * @param value the non-null value to be merged with the existing value
     *        associated with the key or, if no existing value or a null value
     *        is associated with the key, to be associated with the key
     * @param remappingFunction the function to recompute a value if present
     * @return the new value associated with the specified key, or null if no
     *         value is associated with the key
     * @throws UnsupportedOperationException if the {@code put} operation
     *         is not supported by this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws ClassCastException if the class of the specified key or value
     *         prevents it from being stored in this map
     *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified key is null and this map
     *         does not support null keys or the value or remappingFunction is
     *         null
     * @since 1.8
     */
    default V merge(K key, V value,
            BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        Objects.requireNonNull(value);
        V oldValue = get(key);
        V newValue = (oldValue == null) ? value :
                   remappingFunction.apply(oldValue, value);
        if(newValue == null) {
            remove(key);
        } else {
            put(key, newValue);
        }
        return newValue;
    }
}

```