```java 

package java.util;

/**
 * {@link Map}进一步提供其键的总排序。 地图是根据其密钥的
 * {@linkplain Comparable natural ordering}或通常在排序地图创建时提供的
 * {@link Comparator}进行排序的。 迭代排序映射的集合视图
 * （由{@code entrySet}，{@code keySet}和{@code values}方法返回）时会反
 * 映此顺序。 提供了几个额外的操作以利用订购。 （此界面是
 * {@link SortedSet}的地图模拟。）
 *
 * 插入到有序映射中的所有键必须实现{@code Comparable}接口（或由指定的比
 * 较器接受）。 此外，所有这些密钥必须是可相互比较的：
 * {@ code k1.compareTo（k2）}（或{@code comparator.compare（k1，k2）}）
 * 不得为任何密钥抛出{@code ClassCastException} {@code k1}和{@code k2}在
 * 有序地图中。 尝试违反此限制将导致违规方法或构造函数调用抛出
 * {@code ClassCastException}。
 *
 * 请注意，如果有序映射要正确实现{@code Map}接口，则由有序映射维护的排序
 *（无论是否提供显式比较器）必须与equals一致。 （请参阅{@code Comparable}
 * 接口或{@code Comparator}接口，以获得与equals一致的精确定义。）这是因
 * 为{@code Map}接口是根据{@code equals}操作定义的， 但是有序映射使用其
 * {@code compareTo}（或{@code compare}）方法执行所有密钥比较，因此从排
 * 序映射的角度来看，通过此方法认为相等的两个键是相等的。 
 * 树图的行为即使其排序与equals不一致也是明确定义的; 它只是不遵守
 * {@code Map}界面的一般合同。
 *
 * 所有通用的有序映射实现类都应该提供四个“标准”构造函数。 
 * 虽然接口无法指定所需的构造函数，但无法强制执行此建议。
 * 所有有序映射实现的预期“标准”构造函数是：
 * 
 * 1.一个void（无参数）构造函数，它根据键的自然顺序创建一个空的有序映射。
 * 2.具有{@code Comparator}类型的单个参数的构造函数，它创建根据指定的比
 * 较器排序的空的有序映射。
 * 3. 具有{@code Map}类型的单个参数的构造函数，它创建一个具有与其参数相
 * 同的键 - 值映射的新映射，并根据键的自然顺序进行排序。
 * 4.具有{@code SortedMap}类型的单个参数的构造函数，它创建一个新的有
 * 序映射，其具有相同的键 - 值映射和与输入有序映射相同的顺序。
 *
 * 注意：有几种方法返回带有受限键范围的子图。 
 * 这样的范围是半开放的，即它们包括它们的低端点但不包括它们的高端点（如
 * 果适用）。 如果您需要一个封闭范围（包括两个端点），并且密钥类型允许计
 * 算给定密钥的后继，只需从{@code lowEndpoint}请求子范围到
 * {@code successor（highEndpoint）}。 例如，假设{@code m}是一个键是字符
 * 串的映射。 以下习惯用法获取一个视图，其中包含{@code m}中所有键值映射，
 * 其键位于{@code low}和{@code high}之间，包括：
 *   SortedMap&lt;String, V&gt; sub = m.subMap(low, high+"\0");</pre>
 *
 * 可以使用类似的技术来生成开放范围（其中既不包含端点）。 
 * 以下习惯用法获取一个视图，其中包含{@code m}中所有键值映射，其键位于
 * {@code low}和{@code high}之间，不包括：
 *   SortedMap&lt;String, V&gt; sub = m.subMap(low+"\0", high);</pre>
 *
 * <p>This interface is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 *
 * @author  Josh Bloch
 * @see Map
 * @see TreeMap
 * @see SortedSet
 * @see Comparator
 * @see Comparable
 * @see Collection
 * @see ClassCastException
 * @since 1.2
 */

public interface SortedMap<K,V> extends Map<K,V> {
    /**
     * 返回用于对此映射中的键进行排序的比较器，如果此映射使用其键的
     * {@linkplain Comparable natural ordering}，则返回{@code null}。
     *
     * @return the comparator used to order the keys in this map,
     *         or {@code null} if this map uses the natural ordering
     *         of its keys
     */
    Comparator<? super K> comparator();

    /**
     * 返回此映射部分的视图，其键的范围从{@code fromKey}（包括）到
     * {@code toKey}，不包括。 （如果{@code fromKey}和{@code toKey}相等，则
     * 返回的地图为空。）返回的地图由此地图支持，因此返回地图中的更改将反映
     * 在此地图中，反之亦然。 返回的地图支持此地图支持的所有可选地图操作。
     *
     * 返回的映射将在尝试在其范围之外插入键时抛出{@code IllegalArgumentException}。
     *
     * @param fromKey low endpoint (inclusive) of the keys in the returned map
     * @param toKey high endpoint (exclusive) of the keys in the returned map
     * @return a view of the portion of this map whose keys range from
     *         {@code fromKey}, inclusive, to {@code toKey}, exclusive
     * @throws ClassCastException if {@code fromKey} and {@code toKey}
     *         cannot be compared to one another using this map's comparator
     *         (or, if the map has no comparator, using natural ordering).
     *         Implementations may, but are not required to, throw this
     *         exception if {@code fromKey} or {@code toKey}
     *         cannot be compared to keys currently in the map.
     * @throws NullPointerException if {@code fromKey} or {@code toKey}
     *         is null and this map does not permit null keys
     * @throws IllegalArgumentException if {@code fromKey} is greater than
     *         {@code toKey}; or if this map itself has a restricted
     *         range, and {@code fromKey} or {@code toKey} lies
     *         outside the bounds of the range
     */
    SortedMap<K,V> subMap(K fromKey, K toKey);

    /**
     * 返回此映射部分的视图，其键严格小于{@code toKey}。 
     * 返回的地图由此地图支持，因此返回的地图中的更改将反映在此地图中，反之亦然。 
     * 返回的地图支持此地图支持的所有可选地图操作。
     *
     * 返回的映射将在尝试在其范围之外插入键时抛出{@code IllegalArgumentException}。
     *
     * @param toKey high endpoint (exclusive) of the keys in the returned map
     * @return a view of the portion of this map whose keys are strictly
     *         less than {@code toKey}
     * @throws ClassCastException if {@code toKey} is not compatible
     *         with this map's comparator (or, if the map has no comparator,
     *         if {@code toKey} does not implement {@link Comparable}).
     *         Implementations may, but are not required to, throw this
     *         exception if {@code toKey} cannot be compared to keys
     *         currently in the map.
     * @throws NullPointerException if {@code toKey} is null and
     *         this map does not permit null keys
     * @throws IllegalArgumentException if this map itself has a
     *         restricted range, and {@code toKey} lies outside the
     *         bounds of the range
     */
    SortedMap<K,V> headMap(K toKey);

    /**
     * 返回此映射的部分视图，其键大于或等于{@code fromKey}。
     * 返回的地图由此地图支持，因此返回的地图中的更改将反映在此地图中，反之
     * 亦然。 返回的地图支持此地图支持的所有可选地图操作。
     *
     * 返回的映射将在尝试在其范围之外插入键时抛出{@code IllegalArgumentException}。
     *
     * @param fromKey low endpoint (inclusive) of the keys in the returned map
     * @return a view of the portion of this map whose keys are greater
     *         than or equal to {@code fromKey}
     * @throws ClassCastException if {@code fromKey} is not compatible
     *         with this map's comparator (or, if the map has no comparator,
     *         if {@code fromKey} does not implement {@link Comparable}).
     *         Implementations may, but are not required to, throw this
     *         exception if {@code fromKey} cannot be compared to keys
     *         currently in the map.
     * @throws NullPointerException if {@code fromKey} is null and
     *         this map does not permit null keys
     * @throws IllegalArgumentException if this map itself has a
     *         restricted range, and {@code fromKey} lies outside the
     *         bounds of the range
     */
    SortedMap<K,V> tailMap(K fromKey);

    /**
     * 返回此映射中当前的第一个（最低）键。
     *
     * @return the first (lowest) key currently in this map
     * @throws NoSuchElementException if this map is empty
     */
    K firstKey();

    /**
     * 返回此映射中当前的最后一个（最高）键。
     *
     * @return the last (highest) key currently in this map
     * @throws NoSuchElementException if this map is empty
     */
    K lastKey();

    /**
     * 返回此映射中包含的键的{@link Set}视图。 set的迭代器按升序返回键。 
     * 该集由地图支持，因此对地图的更改将反映在集中，反之亦然。 
     * 如果在对集合进行迭代时修改了映射（除非通过迭代器自己的
     * {@code remove}操作），则迭代的结果是未定义的。 该集支持元素删除，
     * 它通过{@code Iterator.remove}，{@ code Set.remove}，
     * {@ code removeAll}，{@ code retainAll}和{@code clear从地图中删除
     * 相应的映射。 操作。 它不支持{@code add}或{@code addAll}操作。
     *
     * @return a set view of the keys contained in this map, sorted in
     *         ascending order
     */
    Set<K> keySet();

    /**
     * 返回此映射中包含的值的{@link Collection}视图。 集合的迭代器以相应键的
     * 升序返回值。 该集合由地图支持，因此对地图的更改将反映在集合中，反之亦然。 
     * 如果在对集合进行迭代时修改了映射（除了通过迭代器自己的
     * {@code remove}操作），迭代的结果是未定义的。 该集合支持元素删除，它通过
     * {@code Iterator.remove}，{@ code Collection.remove}，{@ code removeAll}，
     * {@ code retainAll}和{@code clear}从地图中删除相应的映射。操作。 它不支
     * 持{@code add}或{@code addAll}操作。
     *
     * @return a collection view of the values contained in this map,
     *         sorted in ascending key order
     */
    Collection<V> values();

    /**
     * 返回此映射中包含的映射的{@link Set}视图。 set的迭代器以升序键顺序返回条目。 
     * 该集由地图支持，因此对地图的更改将反映在集中，反之亦然。 
     * 如果在对集合进行迭代时修改了映射（除了通过迭代器自己的
     * {@code remove}操作，或者通过迭代器返回的映射条目上的
     * {@code setValue}操作），迭代的结果 未定义。 该集支持元素删除，它通过
     * {@code Iterator.remove}，{@ code Set.remove}，{@ code removeAll}，
     * {@ code retainAll}和{@code clear}从地图中删除相应的映射。操作。 它不支持
     * {@code add}或{@code addAll}操作。
     *
     * @return a set view of the mappings contained in this map,
     *         sorted in ascending key order
     */
    Set<Map.Entry<K, V>> entrySet();
}

```