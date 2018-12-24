```java
package java.util;

import java.util.function.Predicate;
import java.util.stream.Stream;
import java.util.stream.StreamSupport;

/**
 * 集合中的根接口。 集合表示一组对象，这些对象称为元素。 有些集合允许重复 
* 元素而其他集合则不允许。 有些是有序的，有些是无序的。 JDK不提供此接口 
* 的任何直接实现：它提供了更具体的子接口（如Set和List）的实现。 此接口通
* 常用于传递集合(引用)并可以设计一些所有集合通用的接口。
 *
 * 包或多个集合（可能包含重复元素的无序集合）应直接实现此接口
 *
 * 所有通用Collection实现类（通常通过其子接口间接实现Collection）应提供两
 * 个“标准”构造函数：void（无参数）构造函数，它创建一个空集合，以及一
 * 个构造函数，其中包含一个 Collection类型的参数，使用与其参数相同的元素
 * 创建新集合。 实际上，后一个构造函数允许用户复制任何集合，从而生成所需
 * 实现类型的等效集合。 没有办法强制执行此约定（因为接口不能包含构造函
 * 数），但Java平台库中的所有通用Collection实现都符合。
 *
 * 接口有"破坏"方法, 这个方法修改它们正在操作的集合， 如果这个集合不支持操
 * 作会抛出UnsupportedOperationException。 如果是这种情况，如果调用对
 * 集合没有影响，则这些方法可能（但不是必须）抛出 
 * UnsupportedOperationException。 例如，如果要添加的集合为空，则可以
 * （但不是必须）在不可修改的集合上调用{@link #addAll（Collection）}方法
 * 抛出异常。
 *
 * 某些集合实现对它们可能包含的元素有限制。 例如，某些实现禁止null元素，
 * 并且一些实现对其元素的类型有限制。 尝试添加不合格的元素会引发未经检查
 * 的异常，通常是NullPointerException或ClassCastException。 试图查询不合
 * 格元素的存在可能会引发异常，或者它可能只是返回false; 一些实现将展示前一
 * 种行为，一些将展示后者。 更一般地，尝试对不合格的元素进行操作，其完成
 * 不会导致将不合格的元素插入到集合中，可以在实现的选择中抛出异常或者它
 * 可以成功。 此类异常在此接口的规范中标记为“optional(可选)”。
 *
 * 由每个集合决定自己的同步策略。 在实现没有更强的保证的情况下，奇怪的输
 * 出可能是由于另一个线程正在突然修改集合导致的。 这包括直接调用，将集合
 * 传递给可能执行调用的方法，以及使用现有迭代器来遍历集合。
 *
 * Collections Framework接口中的许多方法都是根据{@link Object＃equals
 *（Object）equals}方法定义的。
 * 例如，{@link #contains（Object）contains（Object o）}方法的规范说：
 *“当且仅当此集合包含至少一个元素e时才返回true
 *（o == null？e == null：    o.equals（e））。“不应将此规范解释为暗示
 * 使用非null参数o调用 Collection.contains将导致为任何元素e调用o.equals（e）。
 * 实现可以自由地实现优化，从而避免等于调用，例如，通过首先比较两个元素的哈希码。
 * （{@link Object＃hashCode（）}规范保证具有不等哈希码的两个对象不能相等。）
 * 更一般地，在实现者认为合适的地方, 各种Collections Framework接口的实现可以
 * 自由地利用底层{@link Object}的指定行为。
 *
 * 一些递归遍历集合的集合操作可能会失败，因为集合直接或间接包含自身的自引用实例。
 * 这包括{@code clone（）}，{@ code equals（）}，{@ code hashCode（）}和
 * {@code toString（）}方法。 实现可以可选地处理自引用场景，但是大多数当前实现
 * 不这样做。
 *
 * <p>This interface is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @implSpec
 * 默认方法实现（继承或其他方式）不应用任何同步协议。 如果{@code Collection}实现
 * 具有特定的同步协议，则它必须覆盖默认实现以应用该协议。
 *
 * @param <E> the type of elements in this collection
 *
 * @author  Josh Bloch
 * @author  Neal Gafter
 * @see     Set
 * @see     List
 * @see     Map
 * @see     SortedSet
 * @see     SortedMap
 * @see     HashSet
 * @see     TreeSet
 * @see     ArrayList
 * @see     LinkedList
 * @see     Vector
 * @see     Collections
 * @see     Arrays
 * @see     AbstractCollection
 * @since 1.2
 */

public interface Collection<E> extends Iterable<E> {
    // Query Operations

    /**
     * 返回此集合中的元素个数。 如果此集合包含多于Integer.MAX_VALUE的元素，
     * 则返回Integer.MAX_VALUE。
     *
     * @return the number of elements in this collection
     */
    int size();

    /**
     * 如果此collection不包含任何元素，则返回true
     *
     * @return <tt>true</tt> if this collection contains no elements
     */
    boolean isEmpty();

    /**
     * 如果此collection包含指定的元素，则返回true。 更正式地，当且仅当此集合
     * 包含至少一个元素e时才返回true（o == null; e == null; o.equals（e））。
     *
     * @param o element whose presence in this collection is to be tested
     * @return <tt>true</tt> if this collection contains the specified
     *         element
     * @throws ClassCastException if the type of the specified element
     *         is incompatible with this collection
     *         (<a href="#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified element is null and this
     *         collection does not permit null elements
     *         (<a href="#optional-restrictions">optional</a>)
     */
    boolean contains(Object o);

    /**
     * 返回此集合中元素的迭代器。 对于返回元素的顺序没有任何保证（除非此集合是
     * 某个提供保证的类的实例）
     *
     * @return an <tt>Iterator</tt> over the elements in this collection
     */
    Iterator<E> iterator();

    /**
     * 返回包含此集合中所有元素的数组。 如果此集合对其迭代器返回的元素的顺序
     * 做出任何保证，则此方法必须以相同的顺序返回元素。
     *
     *  返回的数组将是“安全的”，因为此集合不维护对它的引用。 （换句话说，即使此
     * 集合由数组支持，此方法也必须分配新数组）。 因此调用者可以自由修改返回的数组。
     * （就是只是拷贝元素内容不是直接返回引用）
     *
     * 此方法充当基于阵列和基于集合的API之间的桥梁。
     *
     * @return an array containing all of the elements in this collection
     */
    Object[] toArray();

    /**
     * 返回一个包含此collection中所有元素的数组; 返回数组的运行时类型是指定数组的
     * 运行时类型。 如果集合适合指定的数组，则返回其中。 否则，将使用指定数组的
     * 运行时类型和此集合的大小分配新数组。
     *
     * 如果此集合适合具有备用空间的指定数组（即，数组具有比此集合更多的元素），
     * 则紧跟集合结尾的数组中的元素将设置为null。 （仅当调用者知道此集合不包含任
     * 何null元素时，这在确定此集合的长度时有用。）
     *
     * 如果此集合对其迭代器返回的元素的顺序做出任何保证，则此方法必须以相同的顺序
     * 返回元素。
     *
     * 与{@link #toArray（）}方法一样，此方法充当基于数组的API和基于集合的API之
     * 间的桥梁。 此外，该方法允许精确控制输出阵列的运行时类型，并且在某些情况下
     * 可以用于节省内存分配成本。
     *
     *  假设已知x是一个只包含string的一个集合。下面代码可以用来dump集合到一个
     * 新分配的String数组里面
     *
     * <pre>
     *     String[] y = x.toArray(new String[0]);</pre>
     *
     * Note that <tt>toArray(new Object[0])</tt> is identical in function to
     * <tt>toArray()</tt>.
     *
     * @param <T> the runtime type of the array to contain the collection
     * @param a the array into which the elements of this collection are to be
     *        stored, if it is big enough; otherwise, a new array of the same
     *        runtime type is allocated for this purpose.
     * @return an array containing all of the elements in this collection
     * @throws ArrayStoreException if the runtime type of the specified array
     *         is not a supertype of the runtime type of every element in
     *         this collection
     * @throws NullPointerException if the specified array is null
     */
    <T> T[] toArray(T[] a);

    // Modification Operations

    /**
     * 确保此集合包含指定的元素（可选操作）。 如果此集合因调用而更改，
     * 则返回true。 （如果此集合不允许重复并且已包含指定的元素，则返回false。）
     *
     * 支持此操作的集合可能会限制可能添加到此集合的元素。 特别是，某些集合将拒
     * 绝添加null元素，而其他集合将对可能添加的元素类型施加限制。 集合类应在其
     * 文档中明确指出可以添加哪些元素的任何限制。
     *
     * 如果一个集合因为已经包含该元素的原因而拒绝添加特定元素，那么它必须抛出
     * 异常（而不是返回false）。 这保留了在此调用返回后集合始终包含指定元素的
     * 不变量。
     *
     * @param e element whose presence in this collection is to be ensured
     * @return <tt>true</tt> if this collection changed as a result of the
     *         call
     * @throws UnsupportedOperationException if the <tt>add</tt> operation
     *         is not supported by this collection
     * @throws ClassCastException if the class of the specified element
     *         prevents it from being added to this collection
     * @throws NullPointerException if the specified element is null and this
     *         collection does not permit null elements
     * @throws IllegalArgumentException if some property of the element
     *         prevents it from being added to this collection
     * @throws IllegalStateException if the element cannot be added at this
     *         time due to insertion restrictions
     */
    boolean add(E e);

    /**
     * 从此集合中移除指定元素的单个实例（如果存在）（可选操作）。 更正式地，如
     * 果此集合包含一个或多个此类元素，则删除元素e（o == null？e == null：
     * o.equals（e））。 如果此collection包含指定的元素，则返回true（或如果此集
     * 合因这次调用而更改）。
     *
     * @param o element to be removed from this collection, if present
     * @return <tt>true</tt> if an element was removed as a result of this call
     * @throws ClassCastException if the type of the specified element
     *         is incompatible with this collection
     *         (<a href="#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified element is null and this
     *         collection does not permit null elements
     *         (<a href="#optional-restrictions">optional</a>)
     * @throws UnsupportedOperationException if the <tt>remove</tt> operation
     *         is not supported by this collection
     */
    boolean remove(Object o);


    // Bulk Operations

    /**
     * 如果此collection包含指定collection中的所有元素，则返回true。
     *
     * @param  c collection to be checked for containment in this collection
     * @return <tt>true</tt> if this collection contains all of the elements
     *         in the specified collection
     * @throws ClassCastException if the types of one or more elements
     *         in the specified collection are incompatible with this
     *         collection
     *         (<a href="#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified collection contains one
     *         or more null elements and this collection does not permit null
     *         elements
     *         (<a href="#optional-restrictions">optional</a>),
     *         or if the specified collection is null.
     * @see    #contains(Object)
     */
    boolean containsAll(Collection<?> c);

    /**
     * 将指定集合中的所有元素添加到此集合中（可选操作）。 如果在操作正在进行时
     * 修改了指定的集合，则此操作的行为是不确定的。 （这意味着如果指定的集合是、
     * 此集合，则此调用的行为是未定义的，并且此集合是非空的。）
     *
     * @param c collection containing elements to be added to this collection
     * @return <tt>true</tt> if this collection changed as a result of the call
     * @throws UnsupportedOperationException if the <tt>addAll</tt> operation
     *         is not supported by this collection
     * @throws ClassCastException if the class of an element of the specified
     *         collection prevents it from being added to this collection
     * @throws NullPointerException if the specified collection contains a
     *         null element and this collection does not permit null elements,
     *         or if the specified collection is null
     * @throws IllegalArgumentException if some property of an element of the
     *         specified collection prevents it from being added to this
     *         collection
     * @throws IllegalStateException if not all the elements can be added at
     *         this time due to insertion restrictions
     * @see #add(Object)
     */
    boolean addAll(Collection<? extends E> c);

    /**
     * 删除此集合的所有元素，这些元素也包含在指定的集合中（可选操作）。 此调用返
     * 回后，此集合将不包含与指定集合相同的元素。
     *
     * @param c collection containing elements to be removed from this collection
     * @return <tt>true</tt> if this collection changed as a result of the
     *         call
     * @throws UnsupportedOperationException if the <tt>removeAll</tt> method
     *         is not supported by this collection
     * @throws ClassCastException if the types of one or more elements
     *         in this collection are incompatible with the specified
     *         collection
     *         (<a href="#optional-restrictions">optional</a>)
     * @throws NullPointerException if this collection contains one or more
     *         null elements and the specified collection does not support
     *         null elements
     *         (<a href="#optional-restrictions">optional</a>),
     *         or if the specified collection is null
     * @see #remove(Object)
     * @see #contains(Object)
     */
    boolean removeAll(Collection<?> c);

    /**
     * Removes all of the elements of this collection that satisfy the given
     * predicate.  Errors or runtime exceptions thrown during iteration or by
     * the predicate are relayed to the caller.
     *
     * @implSpec
     * The default implementation traverses all elements of the collection using
     * its {@link #iterator}.  Each matching element is removed using
     * {@link Iterator#remove()}.  If the collection's iterator does not
     * support removal then an {@code UnsupportedOperationException} will be
     * thrown on the first matching element.
     *
     * @param filter a predicate which returns {@code true} for elements to be
     *        removed
     * @return {@code true} if any elements were removed
     * @throws NullPointerException if the specified filter is null
     * @throws UnsupportedOperationException if elements cannot be removed
     *         from this collection.  Implementations may throw this exception if a
     *         matching element cannot be removed or if, in general, removal is not
     *         supported.
     * @since 1.8
     */
    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }

    /**
     * 仅保留此集合中包含在指定集合中的元素（可选操作）。 换句话说，从此集合
     * 中删除未包含在指定集合中的所有元素。
     *
     * @param c collection containing elements to be retained in this collection
     * @return <tt>true</tt> if this collection changed as a result of the call
     * @throws UnsupportedOperationException if the <tt>retainAll</tt> operation
     *         is not supported by this collection
     * @throws ClassCastException if the types of one or more elements
     *         in this collection are incompatible with the specified
     *         collection
     *         (<a href="#optional-restrictions">optional</a>)
     * @throws NullPointerException if this collection contains one or more
     *         null elements and the specified collection does not permit null
     *         elements
     *         (<a href="#optional-restrictions">optional</a>),
     *         or if the specified collection is null
     * @see #remove(Object)
     * @see #contains(Object)
     */
    boolean retainAll(Collection<?> c);

    /**
     * 从此集合中删除所有元素（可选操作）。 此方法返回后，该集合将为空。
     *
     * @throws UnsupportedOperationException if the <tt>clear</tt> operation
     *         is not supported by this collection
     */
    void clear();


    // Comparison and hashing

    /**
     * 集合与指定对象对比是否相等。
     *
     * 虽然Collection接口没有为Object.equals的常规合约添加任何规定，但是
     *“直接”实现Collection接口的程序员（换句话说，创建一个Collection是
     * 一个类但不是Set或List）必须小心 如果他们选择覆盖Object.equals。 没
     * 有必要这样做，最简单的方法是依赖于Object的实现，但实现者可能希望
     * 实现“值比较”来代替默认的“引用比较”。 （List和Set接口要求进行这
     * 样的值比较。）
     *
     * Object.equals方法的一般契约声明equals必须是对称的（换句话说，
     * a.equals（b）当且仅当b.equals（a））。 List.equals和Set.equals的合
     * 同表明列表仅list只会与list相等，set只会与set相等。 因此，当将此集合
     * 与任何列表或集合进行比较时，既不实现List也不实现Set接口的集合类的
     * 自定义equals方法必须返回false。 （同理，不可能编写一个正确既实现
     * Set又实现List接口的类。）
     *
     * @param o object to be compared for equality with this collection
     * @return <tt>true</tt> if the specified object is equal to this
     * collection
     *
     * @see Object#equals(Object)
     * @see Set#equals(Object)
     * @see List#equals(Object)
     */
    boolean equals(Object o);

    /**
     * 返回此集合的哈希码值。 虽然Collection接口没有为Object.hashCode
     * 方法的常规合约添加任何规定，但程序员应注意，任何覆盖
     * Object.equals方法的类也必须覆盖Object.hashCode方法以满足
     * Object的常规协定。 Object.hashCode方法。 特别是，
     * c1.equals（c2）意味着c1.hashCode（）== c2.hashCode（）。
     *
     * @return the hash code value for this collection
     *
     * @see Object#hashCode()
     * @see Object#equals(Object)
     */
    int hashCode();

    /**
     * 在此集合中的元素上创建{@link Spliterator}。
     *
     * 实现应记录spliterator返回的特征值。 如果spliterator返回
     * {@link Spliterator＃SIZED}并且此集合不包含任何元素，则不需要报告此类值。
     *
     * 应该可以返回更高效的spliterator的子类重写默认实现。 为了保留
     * {@link #stream（）}和{@link #parallelStream（）}}方法的预期懒惰
     * 行为，分词符应该具有{@code IMMUTABLE}或{@code CONCURRENT}
     * 的特征，或者是 <a href="Spliterator.html#binding">延迟绑定</a>如
     * 果这些都不实用，那么重写类应该描述spliterator记录的绑定和结构干扰策
     * 略，并且应该覆盖{@link #stream （）}和{@link #parallelStream（）}
     * 方法使用spliterator的{@code Supplier}创建流，如下所示：
     * <pre>{@code
     *     Stream<E> s = StreamSupport.stream(() -> spliterator(), 
     * spliteratorCharacteristics)
     * }</pre>
     * 
     * 这些要求确保{@link #stream（）}和{@link #parallelStream（）}方法
     * 生成的流将在启动终端流操作时反映集合的内容。.
     *
     * @implSpec
     * 默认实现从集合的{@code Iterator}创建<a href="Spliterator.html#binding">
     * 后期绑定</a>spliterator。 spliterator继承了集合迭代器的fail-fast属性。 
     *
     * 创建的{@code Spliterator}报告{@link Spliterator＃SIZED}。
     *
     * @implNote
     * 创建的{@code Spliterator}还会报告{@link Spliterator＃SUBSIZED}。
     *
     * 如果spliterator不包含任何元素，那么除了{@code SIZED}和{@code SUBSIZED}
     * 之外，报告其他特征值不会帮助客户端控制，专门化或简化计算。 但是，这确实可
     * 以为空集合共享使用不可变和空的spliterator实例（请参阅{@link Spliterators＃
     * emptySpliterator（）}），并使客户端能够确定这样的spliterator是否不包含任
     * 何元素。
     *
     * @return a {@code Spliterator} over the elements in this collection
     * @since 1.8
     */
    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 0);
    }

    /**
     * 返回以此集合为源的顺序{@code Stream}。
     *
     * 当{@link #spliterator（）}方法无法返回{@code IMMUTABLE}，{@code
     * CONCURRENT}或后期绑定的spliterator 时，应该重写此方法。 （有关详细
     * 信息，请参阅{@link #spliterator（）}。）
     *
     * @implSpec
     * 默认实现是集合的{@code Spliterator}创建顺序{@code Stream}。
     *
     * @return a sequential {@code Stream} over the elements in this collection
     * @since 1.8
     */
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }

    /**
     * 以此集合为源返回可能并行的{@code Stream}。 此方法允许返回顺序流。
     *
     * 当{@link #spliterator（）}方法无法返回{@code IMMUTABLE}，{@code 
     * CONCURRENT}或后期绑定的spliterator 时，应该重写此方法。 （有关详细
     * 信息，请参阅{@link #spliterator（）}。）
     *
     * @implSpec
     * 默认实现从集合的{@code Spliterator}创建并行{@code Stream}。
     *
     * @return a possibly parallel {@code Stream} over the elements in this
     * collection
     * @since 1.8
     */
    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
}

```