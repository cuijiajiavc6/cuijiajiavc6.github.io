```java 
package java.util;
import java.util.Map.Entry;

/**
 * 此类提供Map接口的骨干实现，以最大限度地减少实现此接口所需的工作量。
 *
 * 要实现不可修改的映射，程序员只需要扩展此类并为entrySet方法提供实现，
 * 该方法返回映射映射的set-view。 通常，返回的集合将依次在AbstractSet上实现。
 * 此set不应支持add或remove方法，并且其迭代器不应支持remove方法。
 *
 * 要实现可修改的映射，程序员必须另外覆盖此类的put方法（否则抛出
 * UnsupportedOperationException），并由entrySet（）返回迭代器.
 * enerator（）必须另外实现其remove方法。
 *
 * 程序员通常应该根据Map接口规范中的建议提供void（无参数）和map构造函数。
 *
 * 此类中每个非抽象方法的文档详细描述了它的实现。 
 * 如果正在实施的地图允许更有效的实施，则可以覆盖这些方法中的每一个。
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 *
 * @author  Josh Bloch
 * @author  Neal Gafter
 * @see Map
 * @see Collection
 * @since 1.2
 */

public abstract class AbstractMap<K,V> implements Map<K,V> {
    /**
     * 唯一的构造函数。 （对于子类构造函数的调用，通常是隐式的。）
     */
    protected AbstractMap() {
    }

    // Query Operations

    /**
     * {@inheritDoc}
     *
     * @implSpec
     * This implementation returns <tt>entrySet().size()</tt>.
     */
    public int size() {
        return entrySet().size();
    }

    /**
     * {@inheritDoc}
     *
     * @implSpec
     * This implementation returns <tt>size() == 0</tt>.
     */
    public boolean isEmpty() {
        return size() == 0;
    }

    /**
     * {@inheritDoc}
     *
     * @implSpec
     * 此实现迭代在entrySet（）上搜索具有指定值的条目。 如果找到这样的条目，
     * 则返回true。 如果迭代在没有找到这样的条目的情况下终止，则返回false。
     * 请注意，此实现需要地图大小的线性时间。
     *
     * @throws ClassCastException   {@inheritDoc}
     * @throws NullPointerException {@inheritDoc}
     */
    public boolean containsValue(Object value) {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        if (value==null) {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (e.getValue()==null)
                    return true;
            }
        } else {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (value.equals(e.getValue()))
                    return true;
            }
        }
        return false;
    }

    /**
     * {@inheritDoc}
     *
     * @implSpec
     * 此实现迭代在entrySet（）上搜索具有指定键的条目。 如果找到这样的条目，则返
     * 回true。 如果迭代在没有找到这样的条目的情况下终止，则返回false。 
     * 请注意，此实现需要地图大小的线性时间; 许多实现都会覆盖此方法。
     *
     * @throws ClassCastException   {@inheritDoc}
     * @throws NullPointerException {@inheritDoc}
     */
    public boolean containsKey(Object key) {
        Iterator<Map.Entry<K,V>> i = entrySet().iterator();
        if (key==null) {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (e.getKey()==null)
                    return true;
            }
        } else {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (key.equals(e.getKey()))
                    return true;
            }
        }
        return false;
    }

    /**
     * {@inheritDoc}
     *
     * @implSpec
     * 此实现迭代在entrySet（）上搜索具有指定键的条目。 如果找到这样的条目，则返
     * 回条目的值。 如果迭代在没有找到这样的条目的情况下终止，则返回null。
     * 请注意，此实现需要地图大小的线性时间; 许多实现都会覆盖此方法。
     *
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     */
    public V get(Object key) {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        if (key==null) {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (e.getKey()==null)
                    return e.getValue();
            }
        } else {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (key.equals(e.getKey()))
                    return e.getValue();
            }
        }
        return null;
    }


    // Modification Operations

    /**
     * {@inheritDoc}
     *
     * @implSpec
     * 此实现始终抛出UnsupportedOperationException。
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     * @throws IllegalArgumentException      {@inheritDoc}
     */
    public V put(K key, V value) {
        throw new UnsupportedOperationException();
    }

    /**
     * {@inheritDoc}
     *
     * @implSpec
     * 此实现迭代在entrySet（）上搜索具有指定键的条目。 如果找到这样的条目，则
     * 使用其getValue操作获取其值，使用迭代器的remove操作从集合（和支持映射）
     * 中删除该条目，并返回保存的值。 如果迭代在没有找到这样的条目的情况下终止，
     * 则返回null。 请注意，此实现需要地图大小的线性时间; 许多实现都会覆盖此方法。
     *
     * 请注意，如果entrySet迭代器不支持remove方法，并且此映射包含指定键的映射，
     * 则此实现会抛出UnsupportedOperationException。
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     */
    public V remove(Object key) {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        Entry<K,V> correctEntry = null;
        if (key==null) {
            while (correctEntry==null && i.hasNext()) {
                Entry<K,V> e = i.next();
                if (e.getKey()==null)
                    correctEntry = e;
            }
        } else {
            while (correctEntry==null && i.hasNext()) {
                Entry<K,V> e = i.next();
                if (key.equals(e.getKey()))
                    correctEntry = e;
            }
        }

        V oldValue = null;
        if (correctEntry !=null) {
            oldValue = correctEntry.getValue();
            i.remove();
        }
        return oldValue;
    }


    // Bulk Operations

    /**
     * {@inheritDoc}
     *
     * @implSpec
     * 此实现迭代指定的map的entrySet（）集合，并为迭代返回的每个条目调用此
     * 映射的put操作一次。
     *
     * 请注意，如果此映射不支持put操作且指定的映射为非空，则此实现将抛出
     * UnsupportedOperationException。
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     * @throws IllegalArgumentException      {@inheritDoc}
     */
    public void putAll(Map<? extends K, ? extends V> m) {
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
            put(e.getKey(), e.getValue());
    }

    /**
     * {@inheritDoc}
     *
     * @implSpec
     * 此实现调用entrySet().clear() 
     *
     * 请注意，如果entrySet不支持clear操作，则此实现会抛出UnsupportedOperationException。
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     */
    public void clear() {
        entrySet().clear();
    }


    // Views

    /**
     * 初始化每个字段以在第一次请求此视图时包含相应视图的实例。
     * 视图是无状态的，因此没有理由创建多个视图。
     *
     * 由于在访问这些字段时没有执行同步，因此预期使用这些字段的
     * java.util.Map视图类没有非最终字段（或除了outer-this之外的任何字段）。 
     * 坚持这一规则将使这些领域的比赛变得良性。
     *
     * 实现也必须只读取一次该字段，如下所示：
     *
     * <pre> {@code
     * public Set<K> keySet() {
     *   Set<K> ks = keySet;  // single racy read
     *   if (ks == null) {
     *     ks = new KeySet();
     *     keySet = ks;
     *   }
     *   return ks;
     * }
     *}</pre>
     */
    transient Set<K>        keySet;
    transient Collection<V> values;

    /**
     * {@inheritDoc}
     *
     * @implSpec
     * 此实现返回一个子类{@link AbstractSet}的集合。 
     * 子类的迭代器方法在此映射的entrySet（）迭代器上返回一个“包装器对象”。
     * size方法委托给这个map的size方法，contains方法委托给这个map的containsKey方法。
     *
     * 该集合在第一次调用此方法时创建，并返回以响应所有后续调用。 
     * 不执行同步，因此对此方法的多次调用很可能不会返回相同的集合。
     */
    public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new AbstractSet<K>() {
                public Iterator<K> iterator() {
                    return new Iterator<K>() {
                        private Iterator<Entry<K,V>> i = entrySet().iterator();

                        public boolean hasNext() {
                            return i.hasNext();
                        }

                        public K next() {
                            return i.next().getKey();
                        }

                        public void remove() {
                            i.remove();
                        }
                    };
                }

                public int size() {
                    return AbstractMap.this.size();
                }

                public boolean isEmpty() {
                    return AbstractMap.this.isEmpty();
                }

                public void clear() {
                    AbstractMap.this.clear();
                }

                public boolean contains(Object k) {
                    return AbstractMap.this.containsKey(k);
                }
            };
            keySet = ks;
        }
        return ks;
    }

    /**
     * {@inheritDoc}
     *
     * @implSpec
     * 此实现返回一个子类{@link AbstractCollection}的集合。 
     * 子类的迭代器方法在此映射的entrySet（）迭代器上返回一个“包装器对象”。 
     * size方法委托给这个map的size方法，contains方法委托给这个map的containsValue方法。
     *
     * 该集合在第一次调用此方法时创建，并在响应所有后续调用时返回。 
     * 没有执行同步，因此对此方法的多次调用很可能不会返回相同的集合。
     */
    public Collection<V> values() {
        Collection<V> vals = values;
        if (vals == null) {
            vals = new AbstractCollection<V>() {
                public Iterator<V> iterator() {
                    return new Iterator<V>() {
                        private Iterator<Entry<K,V>> i = entrySet().iterator();

                        public boolean hasNext() {
                            return i.hasNext();
                        }

                        public V next() {
                            return i.next().getValue();
                        }

                        public void remove() {
                            i.remove();
                        }
                    };
                }

                public int size() {
                    return AbstractMap.this.size();
                }

                public boolean isEmpty() {
                    return AbstractMap.this.isEmpty();
                }

                public void clear() {
                    AbstractMap.this.clear();
                }

                public boolean contains(Object v) {
                    return AbstractMap.this.containsValue(v);
                }
            };
            values = vals;
        }
        return vals;
    }

    public abstract Set<Entry<K,V>> entrySet();


    // Comparison and hashing

    /**
     * 将指定对象与此映射进行比较以获得相等性。 
     * 如果给定对象也是一个映射，并且两个映射表示相同的映射，则返回true。 更
     * 正式地说，如果m1.entrySet（）。equals（m2.entrySet（）），
     * 则两个映射m1和m2表示相同的映射。 这确保了equals方法在Map接口的不同实
     * 现中正常工作。
     *
     * @implSpec
     * 此实现首先检查指定的对象是否是此映射; 如果是这样，它返回true。 
     * 然后，它检查指定的对象是否是一个大小与该映射的大小相同的映射; 如果不是，
     * 则返回false。 如果是这样，它将迭代此映射的entrySet集合，并检查指
     * 定的映射是否包含此映射包含的每个映射。 如果指定的映射未能包含此类映射，
     * 则返回false。 如果迭代完成，则返回true。
     *
     * @param o object to be compared for equality with this map
     * @return <tt>true</tt> if the specified object is equal to this map
     */
    public boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Map))
            return false;
        Map<?,?> m = (Map<?,?>) o;
        if (m.size() != size())
            return false;

        try {
            Iterator<Entry<K,V>> i = entrySet().iterator();
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                K key = e.getKey();
                V value = e.getValue();
                if (value == null) {
                    if (!(m.get(key)==null && m.containsKey(key)))
                        return false;
                } else {
                    if (!value.equals(m.get(key)))
                        return false;
                }
            }
        } catch (ClassCastException unused) {
            return false;
        } catch (NullPointerException unused) {
            return false;
        }

        return true;
    }

    /**
     * 此实现遍历entrySet（），在集合中的每个元素（条目）上调用
     * {@link Map.Entry＃hashCode hashCode（）}，并将结果相加。
     *
     * @implSpec
     * 此实现遍历entrySet（），在集合中的每个元素（条目）上调用
     * {@link Map.Entry＃hashCode hashCode（）}，并将结果相加。
     *
     * @return the hash code value for this map
     * @see Map.Entry#hashCode()
     * @see Object#equals(Object)
     * @see Set#equals(Object)
     */
    public int hashCode() {
        int h = 0;
        Iterator<Entry<K,V>> i = entrySet().iterator();
        while (i.hasNext())
            h += i.next().hashCode();
        return h;
    }

    /**
     * 返回此映射的字符串表示形式。 字符串表示由地图的entrySet视图的迭代器返回
     * 的顺序中的键 - 值映射列表组成，括在大括号（“{}”）中。 
     * 相邻的映射由字符“，”（逗号和空格）分隔。 每个键值映射都呈现为键，后跟等
     * 号（<tt>“=”</ tt>），后跟相关值。 键和值由{@link String＃valueOf
     *（Object）}转换为字符串。
     *
     * @return a string representation of this map
     */
    public String toString() {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        if (! i.hasNext())
            return "{}";

        StringBuilder sb = new StringBuilder();
        sb.append('{');
        for (;;) {
            Entry<K,V> e = i.next();
            K key = e.getKey();
            V value = e.getValue();
            sb.append(key   == this ? "(this Map)" : key);
            sb.append('=');
            sb.append(value == this ? "(this Map)" : value);
            if (! i.hasNext())
                return sb.append('}').toString();
            sb.append(',').append(' ');
        }
    }

    /**
     * 返回此AbstractMap实例的浅表副本：未克隆键和值本身。
     *
     * @return a shallow copy of this map
     */
    protected Object clone() throws CloneNotSupportedException {
        AbstractMap<?,?> result = (AbstractMap<?,?>)super.clone();
        result.keySet = null;
        result.values = null;
        return result;
    }

    /**
     * SimpleEntry和SimpleImmutableEntry的实用方法。 测试是否相等，检查空值。
     *
     * 注意：在解析JDK-8015417之前，不要用Object.equals替换。
     */
    private static boolean eq(Object o1, Object o2) {
        return o1 == null ? o2 == null : o1.equals(o2);
    }

    //实现注意：SimpleEntry和SimpleImmutableEntry是不同的不相关类，即使它们共
    //享一些代码。 由于您无法在子类中添加或减去字段的最终结果，因此它们无法共
    //享表示形式，并且重复代码的数量太小而无法保证公开常见的抽象类。

    /**
     * 保持键和值的条目。 可以使用setValue方法更改该值。 此类有助于构建自定义
     * 地图实现的过程。 例如，在方法Map.entrySet（）。toArray
     * 中返回SimpleEntry实例的数组可能很方便。
     *
     * @since 1.6
     */
    public static class SimpleEntry<K,V>
        implements Entry<K,V>, java.io.Serializable
    {
        private static final long serialVersionUID = -8499721149061103585L;

        private final K key;
        private V value;

        /**
         * 创建表示从指定键到指定值的映射的条目。
         *
         * @param key the key represented by this entry
         * @param value the value represented by this entry
         */
        public SimpleEntry(K key, V value) {
            this.key   = key;
            this.value = value;
        }

        /**
         * 创建表示与指定条目相同的映射的条目。
         *
         * @param entry the entry to copy
         */
        public SimpleEntry(Entry<? extends K, ? extends V> entry) {
            this.key   = entry.getKey();
            this.value = entry.getValue();
        }

        /**
         * 返回与此条目对应的键。
         *
         * @return the key corresponding to this entry
         */
        public K getKey() {
            return key;
        }

        /**
         * 返回与此条目对应的键。
         *
         * @return the value corresponding to this entry
         */
        public V getValue() {
            return value;
        }

        /**
         * 用指定的值替换此条目对应的值。
         *
         * @param value new value to be stored in this entry
         * @return the old value corresponding to the entry
         */
        public V setValue(V value) {
            V oldValue = this.value;
            this.value = value;
            return oldValue;
        }

        /**
         * 将指定对象与此条目进行比较以获得相等性。 如果给定对象也是映射条目，
         * 则返回{@code true}，并且这两个条目表示相同的映射。 更正式地说，两
         * 个条目{@code e1}和{@code e2}代表相同的映射
         * if<pre>
         *   (e1.getKey()==null ?
         *    e2.getKey()==null :
         *    e1.getKey().equals(e2.getKey()))
         *   &amp;&amp;
         *   (e1.getValue()==null ?
         *    e2.getValue()==null :
         *    e1.getValue().equals(e2.getValue()))</pre>
         * 这可确保{@code equals}方法在{@code Map.Entry}接口的不同实现中正常工作。
         *
         * @param o object to be compared for equality with this map entry
         * @return {@code true} if the specified object is equal to this map
         *         entry
         * @see    #hashCode
         */
        public boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            return eq(key, e.getKey()) && eq(value, e.getValue());
        }

        /**
         * 返回此映射条目的哈希码值。 映射条目{@code e}的哈希码定义为：
         *（e.getKey（）== null？0：e.getKey（）。hashCode（））^
         *（e.getValue（）== null？ 0：e.getValue（）。hashCode（））这确
         * 保{@code e1.equals（e2）}暗示{@code e1.hashCode（）== e2.hashCode（）}
         * 对于任何两个条目{@code e1}和{@code e2}，根据
         * {@link Object＃hashCode}的一般合同的要求。
         *
         * @return the hash code value for this map entry
         * @see    #equals
         */
        public int hashCode() {
            return (key   == null ? 0 :   key.hashCode()) ^
                   (value == null ? 0 : value.hashCode());
        }

        /**
         * 返回此映射条目的String表示形式。 
         * 此实现返回此条目的键的字符串表示形式，后跟等号字符
         *（“= <”），后跟此条目值的字符串表示形式。
         *
         * @return a String representation of this map entry
         */
        public String toString() {
            return key + "=" + value;
        }

    }

    /**
     * 保持不可变键和值的Entry。 此类不支持方法setValue。 在返回键
     * - 值映射的线程安全快照的方法中，此类可能很方便。
     *
     * @since 1.6
     */
    public static class SimpleImmutableEntry<K,V>
        implements Entry<K,V>, java.io.Serializable
    {
        private static final long serialVersionUID = 7138329143949025153L;

        private final K key;
        private final V value;

        /**
         * 创建表示从指定键到指定值的映射的条目。
         *
         * @param key the key represented by this entry
         * @param value the value represented by this entry
         */
        public SimpleImmutableEntry(K key, V value) {
            this.key   = key;
            this.value = value;
        }

        /**
         * 创建表示与指定条目相同的映射的条目。
         *
         * @param entry the entry to copy
         */
        public SimpleImmutableEntry(Entry<? extends K, ? extends V> entry) {
            this.key   = entry.getKey();
            this.value = entry.getValue();
        }

        /**
         * 返回与此条目对应的键。
         *
         * @return the key corresponding to this entry
         */
        public K getKey() {
            return key;
        }

        /**
         * 返回与此条目对应的键。
         *
         * @return the value corresponding to this entry
         */
        public V getValue() {
            return value;
        }

        /**
         * 用指定的值替换此条目对应的值（可选操作）。 
         * 此实现只会抛出UnsupportedOperationException，因为此类实现了
         * 不可变的映射条目。
         *
         * @param value new value to be stored in this entry
         * @return (Does not return)
         * @throws UnsupportedOperationException always
         */
        public V setValue(V value) {
            throw new UnsupportedOperationException();
        }

        /**
         * 将指定对象与此条目进行比较以获得相等性。 如果给定对象也是映射条目，
         * 则返回{@code true}，并且这两个条目表示相同的映射。 更正式地说，两
         * 个条目{@code e1}和{@code e2}代表相同的映射
         * if<pre>
         *   (e1.getKey()==null ?
         *    e2.getKey()==null :
         *    e1.getKey().equals(e2.getKey()))
         *   &amp;&amp;
         *   (e1.getValue()==null ?
         *    e2.getValue()==null :
         *    e1.getValue().equals(e2.getValue()))</pre>
         * 这可确保{@code equals}方法在{@code Map.Entry}接口的不同实现中正常工作。
         *
         * @param o object to be compared for equality with this map entry
         * @return {@code true} if the specified object is equal to this map
         *         entry
         * @see    #hashCode
         */
        public boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            return eq(key, e.getKey()) && eq(value, e.getValue());
        }

        /**
         * 返回此映射条目的哈希码值。 映射条目{@code e}的哈希码定义为： 
         *   (e.getKey()==null   ? 0 : e.getKey().hashCode()) ^
         *   (e.getValue()==null ? 0 : e.getValue().hashCode())</pre>
         * This ensures that {@code e1.equals(e2)} implies that
         * {@code e1.hashCode()==e2.hashCode()} for any two Entries
         * {@code e1} and {@code e2}, as required by the general
         * contract of {@link Object#hashCode}.
         *
         * @return the hash code value for this map entry
         * @see    #equals
         */
        public int hashCode() {
            return (key   == null ? 0 :   key.hashCode()) ^
                   (value == null ? 0 : value.hashCode());
        }

        /**
         * 返回此映射条目的String表示形式。 此实现返回此条目的键的字符串表示形式
         * ，后跟等号字符（“=”），后跟此条目值的字符串表示形式。
         *
         * @return a String representation of this map entry
         */
        public String toString() {
            return key + "=" + value;
        }

    }

}


```