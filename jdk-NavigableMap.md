```java  

package java.util;

/**
 * 使用导航方法扩展的{@link SortedMap}返回给定搜索目标的最接近匹配。 
 * 方法{@code lowerEntry}，{@ code floorEntry}，{@ code ceilingEntry}
 * 和{@code higherEntry}返回与密钥相关联的{@code Map.
 * Entry}对象，分别小于，小于或等于，大于或等于 如果没有这样的密钥，
 * 则返回{@code null}并且大于给定密钥。 类似地，方法{@code lowerKey}，
 * {@ code floorKey}，{@ code ceilingKey}和{@code higherKey}仅返回关
 * 联的键。 所有这些方法都是为了定位而不是遍历条目而设计的。
 *
 * 可以按升序或降序键访问和遍历{@code NavigableMap}。
 * {@code descendingMap}方法返回地图视图，其中所有关系和方向方法的感
 * 知都被反转。 升序操作和视图的性能可能比降序操作的速度快。 方法
 * {@code subMap}，{@ code headMap}和{@code tailMap}与同名的
 * {@code SortedMap}方法的不同之处在于接受描述下限和上限是包含还是排
 * 除的其他参数。 任何{@code NavigableMap}的子图必须实现
 * {@code NavigableMap}界面。
 *
 * 此接口还定义了方法{@code firstEntry}，{@ code pollFirstEntry}，
 * {@ code lastEntry}和{@code pollLastEntry}，它们返回和/或删除最
 * 小和最大的映射（如果存在），否则返回{@code 空值}。
 *
 * 入口返回方法的实现应该返回{@code Map.
 * Entry}对，表示生成时映射的快照，因此通常不支持可选的
 * {@code Entry.setValue}方法。 但请注意，可以使用方法{@code put}
 * 更改关联映射中的映射。
 *
 * <p>Methods
 * {@link #subMap(Object, Object) subMap(K, K)},
 * {@link #headMap(Object) headMap(K)}, and
 * {@link #tailMap(Object) tailMap(K)}
 * are specified to return {@code SortedMap} to allow existing
 * implementations of {@code SortedMap} to be compatibly retrofitted to
 * implement {@code NavigableMap}, but extensions and implementations
 * of this interface are encouraged to override these methods to return
 * {@code NavigableMap}.  Similarly,
 * {@link #keySet()} can be overriden to return {@code NavigableSet}.
 *
 * <p>This interface is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @author Doug Lea
 * @author Josh Bloch
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 * @since 1.6
 */
public interface NavigableMap<K,V> extends SortedMap<K,V> {
    /**
     * 返回与严格小于给定键的最大键相关联的键 - 值映射，如果没有这样
     * 的键，则返回{@code null}。
     *
     * @param key the key
     * @return an entry with the greatest key less than {@code key},
     *         or {@code null} if there is no such key
     * @throws ClassCastException if the specified key cannot be compared
     *         with the keys currently in the map
     * @throws NullPointerException if the specified key is null
     *         and this map does not permit null keys
     */
    Map.Entry<K,V> lowerEntry(K key);

    /**
     * 返回严格小于给定键的最大键，如果没有这样的键，则返回{@code null}。
     *
     * @param key the key
     * @return the greatest key less than {@code key},
     *         or {@code null} if there is no such key
     * @throws ClassCastException if the specified key cannot be compared
     *         with the keys currently in the map
     * @throws NullPointerException if the specified key is null
     *         and this map does not permit null keys
     */
    K lowerKey(K key);

    /**
     * 返回与小于或等于给定键的最大键关联的键 - 值映射，如果没有此键，则
     * 返回{@code null}。
     *
     * @param key the key
     * @return an entry with the greatest key less than or equal to
     *         {@code key}, or {@code null} if there is no such key
     * @throws ClassCastException if the specified key cannot be compared
     *         with the keys currently in the map
     * @throws NullPointerException if the specified key is null
     *         and this map does not permit null keys
     */
    Map.Entry<K,V> floorEntry(K key);

    /**
     * 返回小于或等于给定键的最大键，如果没有这样的键，则返回{@code null}。
     *
     * @param key the key
     * @return the greatest key less than or equal to {@code key},
     *         or {@code null} if there is no such key
     * @throws ClassCastException if the specified key cannot be compared
     *         with the keys currently in the map
     * @throws NullPointerException if the specified key is null
     *         and this map does not permit null keys
     */
    K floorKey(K key);

    /**
     * 返回与大于或等于给定键的最小键关联的键 - 值映射，如果没有这样的键，
     * 则返回{@code null}。
     *
     * @param key the key
     * @return an entry with the least key greater than or equal to
     *         {@code key}, or {@code null} if there is no such key
     * @throws ClassCastException if the specified key cannot be compared
     *         with the keys currently in the map
     * @throws NullPointerException if the specified key is null
     *         and this map does not permit null keys
     */
    Map.Entry<K,V> ceilingEntry(K key);

    /**
     * 返回大于或等于给定键的最小键，如果没有这样的键，则返回{@code null}。
     *
     * @param key the key
     * @return the least key greater than or equal to {@code key},
     *         or {@code null} if there is no such key
     * @throws ClassCastException if the specified key cannot be compared
     *         with the keys currently in the map
     * @throws NullPointerException if the specified key is null
     *         and this map does not permit null keys
     */
    K ceilingKey(K key);

    /**
     * 返回与严格大于给定键的最小键关联的键 - 值映射，如果没有这样的键，则
     * 返回{@code null}。
     *
     * @param key the key
     * @return an entry with the least key greater than {@code key},
     *         or {@code null} if there is no such key
     * @throws ClassCastException if the specified key cannot be compared
     *         with the keys currently in the map
     * @throws NullPointerException if the specified key is null
     *         and this map does not permit null keys
     */
    Map.Entry<K,V> higherEntry(K key);

    /**
     * 返回严格大于给定键的最小键，如果没有这样的键，则返回{@code null}。
     *
     * @param key the key
     * @return the least key greater than {@code key},
     *         or {@code null} if there is no such key
     * @throws ClassCastException if the specified key cannot be compared
     *         with the keys currently in the map
     * @throws NullPointerException if the specified key is null
     *         and this map does not permit null keys
     */
    K higherKey(K key);

    /**
     * 返回与此映射中的最小键关联的键 - 值映射，如果映射为空，则返回{@code null}。
     *
     * @return an entry with the least key,
     *         or {@code null} if this map is empty
     */
    Map.Entry<K,V> firstEntry();

    /**
     * 返回与此映射中的最大键关联的键 - 值映射，如果映射为空，则返回{@code null}。
     *
     * @return an entry with the greatest key,
     *         or {@code null} if this map is empty
     */
    Map.Entry<K,V> lastEntry();

    /**
     * 删除并返回与此映射中的最小键关联的键 - 值映射，如果映射为空，则返回
     * {@code null}。
     *
     * @return the removed first entry of this map,
     *         or {@code null} if this map is empty
     */
    Map.Entry<K,V> pollFirstEntry();

    /**
     * 删除并返回与此映射中的最大键关联的键 - 值映射，如果映射为空，则返回
     * {@code null}。
     *
     * @return the removed last entry of this map,
     *         or {@code null} if this map is empty
     */
    Map.Entry<K,V> pollLastEntry();

    /**
     * 返回此映射中包含的映射的逆序视图。 
     * 降序地图由此地图支持，因此对地图的更改将反映在降序地图中，反之亦然。 
     * 如果在对任一映射的集合视图进行迭代时修改了任一映射（除非通过迭代器
     * 自己的{@code remove}操作），则迭代的结果是未定义的。
     *
     * <p>The returned map has an ordering equivalent to
     * <tt>{@link Collections#reverseOrder(Comparator) Collections.reverseOrder}(comparator())</tt>.
     * The expression {@code m.descendingMap().descendingMap()} returns a
     * view of {@code m} essentially equivalent to {@code m}.
     *
     * @return a reverse order view of this map
     */
    NavigableMap<K,V> descendingMap();

    /**
     * 返回此映射中包含的键的{@link NavigableSet}视图。 set的迭代器按升序返回键。
     *  该集由地图支持，因此对地图的更改将反映在集中，反之亦然。 
     * 如果在对集合进行迭代时修改了映射（除非通过迭代器自己的
     * {@code remove}操作），则迭代的结果是未定义的。 该集支持元素删除，它通过
     * {@code Iterator.remove}，{@ code Set.remove}，{@ code removeAll}，
     * {@ code retainAll}和{@code clear从地图中删除相应的映射。 操作。 它不支持
     * {@code add}或{@code addAll}操作。
     *
     * @return a navigable set view of the keys in this map
     */
    NavigableSet<K> navigableKeySet();

    /**
     * 返回此映射中包含的键的反向顺序{@link NavigableSet}视图。
     * set的迭代器按降序返回键。 
     * 该集由地图支持，因此对地图的更改将反映在集中，反之亦然。 
     * 如果在对集合进行迭代时修改了映射（除非通过迭代器自己的
     * {@code remove}操作），则迭代的结果是未定义的。 该集支持元素删除，它
     * 通过{@code Iterator.remove}，{@ code Set.remove}，{@ code removeAll}，
     * {@ code retainAll}和{@code clear从地图中删除相应的映射。 操作。 它不
     * 支持{@code add}或{@code addAll}操作。
     *
     * @return a reverse order navigable set view of the keys in this map
     */
    NavigableSet<K> descendingKeySet();

    /**
     * 返回此映射部分的视图，其键的范围从{@code fromKey}到{@code toKey}。 
     * 如果{@code fromKey}和{@code toKey}相等，则返回的地图为空，除非
     * {@code fromInclusive}和{@code toInclusive}都为真。 
     * 返回的地图由此地图支持，因此返回的地图中的更改将反映在此地图中，反之亦然。
     * 返回的地图支持此地图支持的所有可选地图操作。
     *
     * 返回的映射将在尝试在其范围之外插入键时抛出
     * {@code IllegalArgumentException}，或者构造其端点位于其范围之外的子映射。
     *
     * @param fromKey low endpoint of the keys in the returned map
     * @param fromInclusive {@code true} if the low endpoint
     *        is to be included in the returned view
     * @param toKey high endpoint of the keys in the returned map
     * @param toInclusive {@code true} if the high endpoint
     *        is to be included in the returned view
     * @return a view of the portion of this map whose keys range from
     *         {@code fromKey} to {@code toKey}
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
    NavigableMap<K,V> subMap(K fromKey, boolean fromInclusive,
                             K toKey,   boolean toInclusive);

    /**
     * 返回此映射部分的视图，其键小于（或等于，如果{@code inclusive}为真）
     * {@code toKey}。 返回的地图由此地图支持，因此返回的地图中的更改将反
     * 映在此地图中，反之亦然。 返回的地图支持此地图支持的所有可选地图操作。
     *
     * 返回的映射将在尝试在其范围之外插入键时抛出{@code IllegalArgumentException}。
     *
     * @param toKey high endpoint of the keys in the returned map
     * @param inclusive {@code true} if the high endpoint
     *        is to be included in the returned view
     * @return a view of the portion of this map whose keys are less than
     *         (or equal to, if {@code inclusive} is true) {@code toKey}
     * @throws ClassCastException if {@code toKey} is not compatible
     *         with this map's comparator (or, if the map has no comparator,
     *         if {@code toKey} does not implement {@link Comparable}).
     *         Implementations may, but are not required to, throw this
     *         exception if {@code toKey} cannot be compared to keys
     *         currently in the map.
     * @throws NullPointerException if {@code toKey} is null
     *         and this map does not permit null keys
     * @throws IllegalArgumentException if this map itself has a
     *         restricted range, and {@code toKey} lies outside the
     *         bounds of the range
     */
    NavigableMap<K,V> headMap(K toKey, boolean inclusive);

    /**
     * 返回此映射部分的视图，其键大于（或等于{@code inclusive}为真）
     * {@code fromKey}。 返回的地图由此地图支持，因此返回的地图中
     * 的更改将反映在此地图中，反之亦然。 返回的地图支持此地图支持的
     * 所有可选地图操作。
     *
     * 返回的映射将在尝试在其范围之外插入键时抛出{@code IllegalArgumentException}。
     *
     * @param fromKey low endpoint of the keys in the returned map
     * @param inclusive {@code true} if the low endpoint
     *        is to be included in the returned view
     * @return a view of the portion of this map whose keys are greater than
     *         (or equal to, if {@code inclusive} is true) {@code fromKey}
     * @throws ClassCastException if {@code fromKey} is not compatible
     *         with this map's comparator (or, if the map has no comparator,
     *         if {@code fromKey} does not implement {@link Comparable}).
     *         Implementations may, but are not required to, throw this
     *         exception if {@code fromKey} cannot be compared to keys
     *         currently in the map.
     * @throws NullPointerException if {@code fromKey} is null
     *         and this map does not permit null keys
     * @throws IllegalArgumentException if this map itself has a
     *         restricted range, and {@code fromKey} lies outside the
     *         bounds of the range
     */
    NavigableMap<K,V> tailMap(K fromKey, boolean inclusive);

    /**
     * {@inheritDoc}
     *
     * <p>Equivalent to {@code subMap(fromKey, true, toKey, false)}.
     *
     * @throws ClassCastException       {@inheritDoc}
     * @throws NullPointerException     {@inheritDoc}
     * @throws IllegalArgumentException {@inheritDoc}
     */
    SortedMap<K,V> subMap(K fromKey, K toKey);

    /**
     * {@inheritDoc}
     *
     * <p>Equivalent to {@code headMap(toKey, false)}.
     *
     * @throws ClassCastException       {@inheritDoc}
     * @throws NullPointerException     {@inheritDoc}
     * @throws IllegalArgumentException {@inheritDoc}
     */
    SortedMap<K,V> headMap(K toKey);

    /**
     * {@inheritDoc}
     *
     * <p>Equivalent to {@code tailMap(fromKey, true)}.
     *
     * @throws ClassCastException       {@inheritDoc}
     * @throws NullPointerException     {@inheritDoc}
     * @throws IllegalArgumentException {@inheritDoc}
     */
    SortedMap<K,V> tailMap(K fromKey);
}


```