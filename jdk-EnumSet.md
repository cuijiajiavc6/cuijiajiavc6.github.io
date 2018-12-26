```java 

package java.util;

import sun.misc.SharedSecrets;

/**
 * 用于枚举类型的专用{@link Set}实现。枚举集中的所有元素必须来自单个枚举类型，
 * 该类型在创建集时显式或隐式指定。枚举集在内部表示为位向量。这种表现非常紧凑
 * 和高效。这个类的空间和时间性能应该足够好，以允许它作为传统的基于int的“位标
 * 志”的高质量，类型安全的替代品。即使批量操作（例如containsAll和retainAll）如
 * 果它们的参数也是枚举集，也应该非常快速地运行。
 * 
 * 迭代器方法返回的迭代器以其自然顺序（枚举枚举常量的顺序）遍历元素。返回的迭
 * 代器是弱一致的：它永远不会抛出{@link ConcurrentModificationException}，它
 * 可能会也可能不会显示迭代进行过程中对集合所做的任何修改的影响。
 *
 * 不允许使用空元素。尝试插入null元素将抛出
 * {@link NullPointerException}。但是，尝试测试是否存在null元素或删除一个元素
 * 将正常运行。
 *
 * 与大多数集合实现一样，EnumSet不同步。如果多个线程同时访问枚举集，并且至少有
 * 一个线程修改了该集，则应该在外部进行同步。这通常通过在自然封装枚举集的某个
 * 对象上同步来完成。如果不存在此类对象，则应使用
 * {@link Collections＃synchronizedSet}方法“包装”该集合。这最好在创建时完成，
 * 以防止意外的不同步访问：
 *
 * <pre>
 * Set&lt;MyEnum&gt; s = Collections.synchronizedSet(EnumSet.noneOf(MyEnum.class));
 * </pre>
 *
 * 实施说明：所有基本操作都在恒定时间内执行。 他们很可能（虽然不能保证）比他们
 * 的{@link HashSet}同行快得多。 如果它们的参数也是枚举集，即使批量操作也会在恒定时间内执行。
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @author Josh Bloch
 * @since 1.5
 * @see EnumMap
 * @serial exclude
 */
public abstract class EnumSet<E extends Enum<E>> extends AbstractSet<E>
    implements Cloneable, java.io.Serializable
{
    /**
     * 该集合中所有元素的类。
     */
    final Class<E> elementType;

    /**
     * 包含T的所有值（缓存性能。）
     */
    final Enum<?>[] universe;

    private static Enum<?>[] ZERO_LENGTH_ENUM_ARRAY = new Enum<?>[0];

    EnumSet(Class<E>elementType, Enum<?>[] universe) {
        this.elementType = elementType;
        this.universe    = universe;
    }

    /**
     * 创建具有指定元素类型的空枚举集。
     *
     * @param <E> The class of the elements in the set
     * @param elementType the class object of the element type for this enum
     *     set
     * @return An empty enum set of the specified type.
     * @throws NullPointerException if <tt>elementType</tt> is null
     */
    public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        Enum<?>[] universe = getUniverse(elementType);
        if (universe == null)
            throw new ClassCastException(elementType + " not an enum");

        if (universe.length <= 64)
            return new RegularEnumSet<>(elementType, universe);
        else
            return new JumboEnumSet<>(elementType, universe);
    }

    /**
     * 创建一个包含指定元素类型中所有元素的枚举集。
     *
     * @param <E> The class of the elements in the set
     * @param elementType the class object of the element type for this enum
     *     set
     * @return An enum set containing all the elements in the specified type.
     * @throws NullPointerException if <tt>elementType</tt> is null
     */
    public static <E extends Enum<E>> EnumSet<E> allOf(Class<E> elementType) {
        EnumSet<E> result = noneOf(elementType);
        result.addAll();
        return result;
    }

    /**
     * 将相应枚举类型中的所有元素添加到此枚举集中，该枚举集在调用之前为空。
     */
    abstract void addAll();

    /**
     * 创建一个枚举集，其元素类型与指定的枚举集相同，最初包含相同的元素（如果有）。
     *
     * @param <E> The class of the elements in the set
     * @param s the enum set from which to initialize this enum set
     * @return A copy of the specified enum set.
     * @throws NullPointerException if <tt>s</tt> is null
     */
    public static <E extends Enum<E>> EnumSet<E> copyOf(EnumSet<E> s) {
        return s.clone();
    }

    /**
     * 创建从指定集合初始化的枚举集。 
     * 如果指定的集合是EnumSet实例，则此静态工厂方法的行为与{@link #copyOf（EnumSet）}相同。
     * 否则，指定的集合必须至少包含一个元素（以便确定新的枚举集的元素类型）。
     *
     * @param <E> The class of the elements in the collection
     * @param c the collection from which to initialize this enum set
     * @return An enum set initialized from the given collection.
     * @throws IllegalArgumentException if <tt>c</tt> is not an
     *     <tt>EnumSet</tt> instance and contains no elements
     * @throws NullPointerException if <tt>c</tt> is null
     */
    public static <E extends Enum<E>> EnumSet<E> copyOf(Collection<E> c) {
        if (c instanceof EnumSet) {
            return ((EnumSet<E>)c).clone();
        } else {
            if (c.isEmpty())
                throw new IllegalArgumentException("Collection is empty");
            Iterator<E> i = c.iterator();
            E first = i.next();
            EnumSet<E> result = EnumSet.of(first);
            while (i.hasNext())
                result.add(i.next());
            return result;
        }
    }

    /**
     * 创建一个枚举集，其元素类型与指定的枚举集相同，最初包含此类型中未包含在指定集
     * 中的所有元素。
     *
     * @param <E> The class of the elements in the enum set
     * @param s the enum set from whose complement to initialize this enum set
     * @return The complement of the specified set in this set
     * @throws NullPointerException if <tt>s</tt> is null
     */
    public static <E extends Enum<E>> EnumSet<E> complementOf(EnumSet<E> s) {
        EnumSet<E> result = copyOf(s);
        result.complement();
        return result;
    }

    /**
     * 创建最初包含指定元素的枚举集。
     * 
     * 存在此方法的过载以初始化具有一到五个元素的枚举集。 提供了使用varargs功能
     * 的第六次重载。 这种重载可用于创建最初包含任意数量元素的枚举集，
     * 但可能比不使用varargs的重载运行得慢。
     *
     * @param <E> The class of the specified element and of the set
     * @param e the element that this set is to contain initially
     * @throws NullPointerException if <tt>e</tt> is null
     * @return an enum set initially containing the specified element
     */
    public static <E extends Enum<E>> EnumSet<E> of(E e) {
        EnumSet<E> result = noneOf(e.getDeclaringClass());
        result.add(e);
        return result;
    }

    /**
     * 创建最初包含指定元素的枚举集。
     *
     * 存在此方法的过载以初始化具有一到五个元素的枚举集。 提供了使用varargs
     * 功能的第六次重载。 这种重载可用于创建最初包含任意数量元素的枚举集，
     * 但可能比不使用varargs的重载运行得慢。
     *
     * @param <E> The class of the parameter elements and of the set
     * @param e1 an element that this set is to contain initially
     * @param e2 another element that this set is to contain initially
     * @throws NullPointerException if any parameters are null
     * @return an enum set initially containing the specified elements
     */
    public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2) {
        EnumSet<E> result = noneOf(e1.getDeclaringClass());
        result.add(e1);
        result.add(e2);
        return result;
    }

    /**
     * 创建最初包含指定元素的枚举集。
     * 
     * 存在此方法的过载以初始化具有一到五个元素的枚举集。 提供了使用
     * varargs功能的第六次重载。 这种重载可用于创建最初包含任意数量元素的枚举集，
     * 但可能比不使用varargs的重载运行得慢。
     *
     * @param <E> The class of the parameter elements and of the set
     * @param e1 an element that this set is to contain initially
     * @param e2 another element that this set is to contain initially
     * @param e3 another element that this set is to contain initially
     * @throws NullPointerException if any parameters are null
     * @return an enum set initially containing the specified elements
     */
    public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3) {
        EnumSet<E> result = noneOf(e1.getDeclaringClass());
        result.add(e1);
        result.add(e2);
        result.add(e3);
        return result;
    }

    /**
     * 创建最初包含指定元素的枚举集。
     *
     * 存在此方法的过载以初始化具有一到五个元素的枚举集。 提供了使用varargs功能的
     * 第六次重载。 这种重载可用于创建最初包含任意数量元素的枚举集，但可能比不
     * 使用varargs的重载运行得慢。
     *
     * @param <E> The class of the parameter elements and of the set
     * @param e1 an element that this set is to contain initially
     * @param e2 another element that this set is to contain initially
     * @param e3 another element that this set is to contain initially
     * @param e4 another element that this set is to contain initially
     * @throws NullPointerException if any parameters are null
     * @return an enum set initially containing the specified elements
     */
    public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4) {
        EnumSet<E> result = noneOf(e1.getDeclaringClass());
        result.add(e1);
        result.add(e2);
        result.add(e3);
        result.add(e4);
        return result;
    }

    /**
     * 创建最初包含指定元素的枚举集。
     *
     * 存在此方法的过载以初始化具有一到五个元素的枚举集。 提供了使用varargs功
     * 能的第六次重载。 这种重载可用于创建最初包含任意数量元素的枚举集，
     * 但可能比不使用varargs的重载运行得慢。
     *
     * @param <E> The class of the parameter elements and of the set
     * @param e1 an element that this set is to contain initially
     * @param e2 another element that this set is to contain initially
     * @param e3 another element that this set is to contain initially
     * @param e4 another element that this set is to contain initially
     * @param e5 another element that this set is to contain initially
     * @throws NullPointerException if any parameters are null
     * @return an enum set initially containing the specified elements
     */
    public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4,
                                                    E e5)
    {
        EnumSet<E> result = noneOf(e1.getDeclaringClass());
        result.add(e1);
        result.add(e2);
        result.add(e3);
        result.add(e4);
        result.add(e5);
        return result;
    }

    /**
     * 创建最初包含指定元素的枚举集。 此工厂的参数列表使用varargs功能，可用于创
     * 建最初包含任意数量元素的枚举集，但它可能比不使用varargs的重载运行得慢。
     *
     * @param <E> The class of the parameter elements and of the set
     * @param first an element that the set is to contain initially
     * @param rest the remaining elements the set is to contain initially
     * @throws NullPointerException if any of the specified elements are null,
     *     or if <tt>rest</tt> is null
     * @return an enum set initially containing the specified elements
     */
    @SafeVarargs
    public static <E extends Enum<E>> EnumSet<E> of(E first, E... rest) {
        EnumSet<E> result = noneOf(first.getDeclaringClass());
        result.add(first);
        for (E e : rest)
            result.add(e);
        return result;
    }

    /**
     * 创建一个枚举集，最初包含由两个指定端点定义的范围中的所有元素。 
     * 回的集合将包含端点本身，这些端点可能相同但不得出现故障。
     *
     * @param <E> The class of the parameter elements and of the set
     * @param from the first element in the range
     * @param to the last element in the range
     * @throws NullPointerException if {@code from} or {@code to} are null
     * @throws IllegalArgumentException if {@code from.compareTo(to) > 0}
     * @return an enum set initially containing all of the elements in the
     *         range defined by the two specified endpoints
     */
    public static <E extends Enum<E>> EnumSet<E> range(E from, E to) {
        if (from.compareTo(to) > 0)
            throw new IllegalArgumentException(from + " > " + to);
        EnumSet<E> result = noneOf(from.getDeclaringClass());
        result.addRange(from, to);
        return result;
    }

    /**
     * 将指定范围添加到此枚举集中，该枚举集在调用之前为空。
     */
    abstract void addRange(E from, E to);

    /**
     * Returns a copy of this set.
     *
     * @return a copy of this set
     */
    @SuppressWarnings("unchecked")
    public EnumSet<E> clone() {
        try {
            return (EnumSet<E>) super.clone();
        } catch(CloneNotSupportedException e) {
            throw new AssertionError(e);
        }
    }

    /**
     * 补充此枚举集的内容。
     */
    abstract void complement();

    /**
     * Throws an exception if e is not of the correct type for this enum set.
     */
    final void typeCheck(E e) {
        Class<?> eClass = e.getClass();
        if (eClass != elementType && eClass.getSuperclass() != elementType)
            throw new ClassCastException(eClass + " != " + elementType);
    }

    /**
     * 返回包含E的所有值。结果由所有调用者取消克隆，缓存和共享。
     */
    private static <E extends Enum<E>> E[] getUniverse(Class<E> elementType) {
        return SharedSecrets.getJavaLangAccess()
                                        .getEnumConstantsShared(elementType);
    }

    /**
     * 此类用于序列化所有EnumSet实例，而不管实现类型如何。
     * 它捕获它们的“逻辑内容”，并使用公共静态工厂重建它们。
     * 这对于确保特定实现类型的存在是实现细节是必要的
     *
     * @serial include
     */
    private static class SerializationProxy <E extends Enum<E>>
        implements java.io.Serializable
    {
        /**
         * The element type of this enum set.
         *
         * @serial
         */
        private final Class<E> elementType;

        /**
         * The elements contained in this enum set.
         *
         * @serial
         */
        private final Enum<?>[] elements;

        SerializationProxy(EnumSet<E> set) {
            elementType = set.elementType;
            elements = set.toArray(ZERO_LENGTH_ENUM_ARRAY);
        }

        // instead of cast to E, we should perhaps use elementType.cast()
        // to avoid injection of forged stream, but it will slow the implementation
        @SuppressWarnings("unchecked")
        private Object readResolve() {
            EnumSet<E> result = EnumSet.noneOf(elementType);
            for (Enum<?> e : elements)
                result.add((E)e);
            return result;
        }

        private static final long serialVersionUID = 362491234563181265L;
    }

    Object writeReplace() {
        return new SerializationProxy<>(this);
    }

    // readObject method for the serialization proxy pattern
    // See Effective Java, Second Ed., Item 78.
    private void readObject(java.io.ObjectInputStream stream)
        throws java.io.InvalidObjectException {
        throw new java.io.InvalidObjectException("Proxy required");
    }
}

```