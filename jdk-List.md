```java
package java.util;

import java.util.function.UnaryOperator;

/**
 * 有序集合（也称为序列）。 该接口的用户可以精确控制列表中每个元素的插入位置。 
 * 用户可以通过整数索引（列表中的位置）访问元素，并搜索列表中的元素。
 *
 * 与集合不同，列表通常允许重复元素。 更正式地，列表通常允许元素对e1和e2成为
 * e1.equals（e2），并且如果它们根本允许空元素，则它们通常允许多个空元素。 
 * 有人可能希望通过在用户尝试插入运行时异常时抛出运行时异常来实现禁止重复的列
 * 表，这是不可想象的，但我们希望这种用法很少见。
 *
 * List接口在迭代器，add，remove，equals和hashCode方法的契约上放置了除Collection
 * 接口中指定的规则之外的其他规定。 为方便起见，此处还包括其他继承方法的声明。
 *
 * List接口提供了四种对列表元素进行位置（索引）访问的方法。 列表（如Java数组）
 * 基于零。 注意，对于某些实现（例如，LinkedList类），这些操作可以与索引值成比
 * 例地执行。 因此，如果调用者不知道实现，则迭代列表中的元素通常优选通过它进行
 * 索引。
 *
 * List接口提供了一个特殊的迭代器，称为ListIterator，它允许元素插入和替换，以及
 * Iterator接口提供的常规操作之外的双向访问。 提供了一种方法来获得从列表中的指定
 * 位置开始的列表迭代器。
 *
 * List接口提供了两种搜索指定对象的方法。 从性能的角度来看，应谨慎使用这些方法。 
 * 在许多实现中，它们将执行昂贵的线性搜索。
 *
 * List接口提供了两种方法来有效地在列表中的任意点插入和删除多个元素。
 *
 * 注意：虽然允许列表将自己包含为元素，但建议极其谨慎：equals和hashCode方法不再
 * 在这样的列表中很好地定义。
 *
 * 某些列表实现对它们可能包含的元素有限制。 例如，某些实现禁止null元素，并且一些
 * 实现对其元素的类型有限制。 尝试添加不合格的元素会引发未经检查的异常，通常是
 * NullPointerException或ClassCastException。 试图查询不合格元素的存在可能会引发
 * 异常，或者它可能只是返回false; 一些实现将展示前一种行为，一些将展示后者。 
 * 更一般地，在完成不会导致将不合格元素插入列表的不合格元素上尝试操作可能会引发异
 * 常，或者可能在实现的选择中成功。 此类异常在此接口的规范中标记为“可选”。
 *
 * <p>This interface is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @param <E> the type of elements in this list
 *
 * @author  Josh Bloch
 * @author  Neal Gafter
 * @see Collection
 * @see Set
 * @see ArrayList
 * @see LinkedList
 * @see Vector
 * @see Arrays#asList(Object[])
 * @see Collections#nCopies(int, Object)
 * @see Collections#EMPTY_LIST
 * @see AbstractList
 * @see AbstractSequentialList
 * @since 1.2
 */

public interface List<E> extends Collection<E> {
    // Query Operations

    /**
     * Returns the number of elements in this list.  If this list contains
     * more than <tt>Integer.MAX_VALUE</tt> elements, returns
     * <tt>Integer.MAX_VALUE</tt>.
     *
     * @return the number of elements in this list
     */
    int size();

    /**
     * Returns <tt>true</tt> if this list contains no elements.
     *
     * @return <tt>true</tt> if this list contains no elements
     */
    boolean isEmpty();

    /**
     * Returns <tt>true</tt> if this list contains the specified element.
     * More formally, returns <tt>true</tt> if and only if this list contains
     * at least one element <tt>e</tt> such that
     * <tt>(o==null&nbsp;?&nbsp;e==null&nbsp;:&nbsp;o.equals(e))</tt>.
     *
     * @param o element whose presence in this list is to be tested
     * @return <tt>true</tt> if this list contains the specified element
     * @throws ClassCastException if the type of the specified element
     *         is incompatible with this list
     * (<a href="Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified element is null and this
     *         list does not permit null elements
     * (<a href="Collection.html#optional-restrictions">optional</a>)
     */
    boolean contains(Object o);

    /**
     * Returns an iterator over the elements in this list in proper sequence.
     *
     * @return an iterator over the elements in this list in proper sequence
     */
    Iterator<E> iterator();

    /**
     * 以适当的顺序（从第一个元素到最后一个元素）返回包含此列表中所有元素的数组。
     *
     * 返回的数组将是“安全的”，因为此列表不会保留对它的引用。 （换句话说，即使此列表由
     * 数组支持，此方法也必须分配新数组）。 因此调用者可以自由修改返回的数组。
     *
     * 此方法充当基于阵列和基于集合的API之间的桥梁。
     *
     * @return an array containing all of the elements in this list in proper
     *         sequence
     * @see Arrays#asList(Object[])
     */
    Object[] toArray();

    /**
     * 以适当的顺序返回包含此列表中所有元素的数组（从第一个元素到最后一个元素）; 
     * 返回数组的运行时类型是指定数组的运行时类型。 如果列表适合指定的数组，则返
     * 回其中。 否则，将为新数组分配指定数组的运行时类型和此列表的大小。
     *
     * 如果列表适合指定的数组，并且有空余空间（即，数组的元素多于列表），则紧跟在列
     * 表末尾的数组中的元素将设置为null。 *（仅当调用者知道列表不包含任何null元素时，
     * 这在确定列表长度时很有用。）
     *
     * 与{@link #toArray（）}方法一样，此方法充当基于数组的API和基于集合的API之间的桥梁。
     * 此外，该方法允许精确控制输出阵列的运行时类型，并且在某些情况下可以用于节省内存分
     * 配成本。
     *
     * 假设x是已知仅包含字符串的列表。 以下代码可用于将列表转储到新分配的String数组中：
     *
     * <pre>{@code
     *     String[] y = x.toArray(new String[0]);
     * }</pre>
     *
     * Note that <tt>toArray(new Object[0])</tt> is identical in function to
     * <tt>toArray()</tt>.
     *
     * @param a the array into which the elements of this list are to
     *          be stored, if it is big enough; otherwise, a new array of the
     *          same runtime type is allocated for this purpose.
     * @return an array containing the elements of this list
     * @throws ArrayStoreException if the runtime type of the specified array
     *         is not a supertype of the runtime type of every element in
     *         this list
     * @throws NullPointerException if the specified array is null
     */
    <T> T[] toArray(T[] a);


    // Modification Operations

    /**
     * 将指定的元素追加到此列表的末尾（可选操作）。
     *
     * 支持此操作的列表可能会限制可能添加到此列表的元素。
     * 特别是，某些列表将拒绝添加null元素，而其他列表将对可能添加的元素类型
     * 施加限制。 列表类应在其文档中明确指出可以添加哪些元素的任何限制。
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     * @throws UnsupportedOperationException if the <tt>add</tt> operation
     *         is not supported by this list
     * @throws ClassCastException if the class of the specified element
     *         prevents it from being added to this list
     * @throws NullPointerException if the specified element is null and this
     *         list does not permit null elements
     * @throws IllegalArgumentException if some property of this element
     *         prevents it from being added to this list
     */
    boolean add(E e);

    /**
     * 从该列表中删除指定元素的第一个匹配项（如果存在）（可选操作）。 如果此列表不包含该元素，
     * 则不会更改。 更正式地，删除具有最低索引i的元素，使得
     *（o == null？get（i）== null：o.equals（get（i）））（如果存在这样的元素）。 
     * 如果此列表包含指定的元素，则返回true（或等效地，如果此列表因调用而更改）。
     *
     * @param o element to be removed from this list, if present
     * @return <tt>true</tt> if this list contained the specified element
     * @throws ClassCastException if the type of the specified element
     *         is incompatible with this list
     * (<a href="Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified element is null and this
     *         list does not permit null elements
     * (<a href="Collection.html#optional-restrictions">optional</a>)
     * @throws UnsupportedOperationException if the <tt>remove</tt> operation
     *         is not supported by this list
     */
    boolean remove(Object o);


    // Bulk Modification Operations

    /**
     * 如果此列表包含指定集合的所有元素，则返回true。
     *
     * @param  c collection to be checked for containment in this list
     * @return <tt>true</tt> if this list contains all of the elements of the
     *         specified collection
     * @throws ClassCastException if the types of one or more elements
     *         in the specified collection are incompatible with this
     *         list
     * (<a href="Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified collection contains one
     *         or more null elements and this list does not permit null
     *         elements
     *         (<a href="Collection.html#optional-restrictions">optional</a>),
     *         or if the specified collection is null
     * @see #contains(Object)
     */
    boolean containsAll(Collection<?> c);

    /**
     * 将指定集合中的所有元素按指定集合的迭代器（可选操作）返回的顺序追加到此列表
     * 的末尾。 如果在操作正在进行时修改了指定的集合，则此操作的行为是不确定的。
     *（请注意，如果指定的集合是此列表，则会发生这种情况，并且它是非空的。）
     *
     * @param c collection containing elements to be added to this list
     * @return <tt>true</tt> if this list changed as a result of the call
     * @throws UnsupportedOperationException if the <tt>addAll</tt> operation
     *         is not supported by this list
     * @throws ClassCastException if the class of an element of the specified
     *         collection prevents it from being added to this list
     * @throws NullPointerException if the specified collection contains one
     *         or more null elements and this list does not permit null
     *         elements, or if the specified collection is null
     * @throws IllegalArgumentException if some property of an element of the
     *         specified collection prevents it from being added to this list
     * @see #add(Object)
     */
    boolean addAll(Collection<? extends E> c);

    /**
     * 将指定集合中的所有元素插入到指定位置的此列表中（可选操作）。 
     * 将当前位置的元素（如果有）和任何后续元素向右移动（增加其索引）。 
     * 新元素将按照指定集合的迭代器返回的顺序出现在此列表中。 
     * 如果在操作正在进行时修改了指定的集合，则此操作的行为是不确定的。 
     *（请注意，如果指定的集合是此列表，则会发生这种情况，并且它是非空的。）
     *
     * @param index index at which to insert the first element from the
     *              specified collection
     * @param c collection containing elements to be added to this list
     * @return <tt>true</tt> if this list changed as a result of the call
     * @throws UnsupportedOperationException if the <tt>addAll</tt> operation
     *         is not supported by this list
     * @throws ClassCastException if the class of an element of the specified
     *         collection prevents it from being added to this list
     * @throws NullPointerException if the specified collection contains one
     *         or more null elements and this list does not permit null
     *         elements, or if the specified collection is null
     * @throws IllegalArgumentException if some property of an element of the
     *         specified collection prevents it from being added to this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         (<tt>index &lt; 0 || index &gt; size()</tt>)
     */
    boolean addAll(int index, Collection<? extends E> c);

    /**
     * 从此列表中删除指定集合中包含的所有元素（可选操作）。
     *
     * @param c collection containing elements to be removed from this list
     * @return <tt>true</tt> if this list changed as a result of the call
     * @throws UnsupportedOperationException if the <tt>removeAll</tt> operation
     *         is not supported by this list
     * @throws ClassCastException if the class of an element of this list
     *         is incompatible with the specified collection
     * (<a href="Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if this list contains a null element and the
     *         specified collection does not permit null elements
     *         (<a href="Collection.html#optional-restrictions">optional</a>),
     *         or if the specified collection is null
     * @see #remove(Object)
     * @see #contains(Object)
     */
    boolean removeAll(Collection<?> c);

    /**
     * 仅保留此列表中包含在指定集合中的元素（可选操作）。 换句话说，从该列表中删除未
     * 包含在指定集合中的所有元素。
     *
     * @param c collection containing elements to be retained in this list
     * @return <tt>true</tt> if this list changed as a result of the call
     * @throws UnsupportedOperationException if the <tt>retainAll</tt> operation
     *         is not supported by this list
     * @throws ClassCastException if the class of an element of this list
     *         is incompatible with the specified collection
     * (<a href="Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if this list contains a null element and the
     *         specified collection does not permit null elements
     *         (<a href="Collection.html#optional-restrictions">optional</a>),
     *         or if the specified collection is null
     * @see #remove(Object)
     * @see #contains(Object)
     */
    boolean retainAll(Collection<?> c);

    /**
     * 将该列表的每个元素替换为将运算符应用于该元素的结果。 操作员抛出的错误或运
     * 行时异常将转发给调用者。
     *
     * @implSpec
     * The default implementation is equivalent to, for this {@code list}:
     * <pre>{@code
     *     final ListIterator<E> li = list.listIterator();
     *     while (li.hasNext()) {
     *         li.set(operator.apply(li.next()));
     *     }
     * }</pre>
     *
     * If the list's list-iterator does not support the {@code set} operation
     * then an {@code UnsupportedOperationException} will be thrown when
     * replacing the first element.
     *
     * @param operator the operator to apply to each element
     * @throws UnsupportedOperationException if this list is unmodifiable.
     *         Implementations may throw this exception if an element
     *         cannot be replaced or if, in general, modification is not
     *         supported
     * @throws NullPointerException if the specified operator is null or
     *         if the operator result is a null value and this list does
     *         not permit null elements
     *         (<a href="Collection.html#optional-restrictions">optional</a>)
     * @since 1.8
     */
    default void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        final ListIterator<E> li = this.listIterator();
        while (li.hasNext()) {
            li.set(operator.apply(li.next()));
        }
    }

    /**
     * 根据指定的{@link Comparator}引发的顺序对此列表进行排序。
     *
     * 此列表中的所有元素必须使用指定的比较器进行相互比较
     * （即，{@ code c.compare（e1，e2）}不得为任何元素{@code e1}和{@code抛出
     * {@code ClassCastException} e2}在列表中）。
     *
     * 如果指定的比较器为{@code null}，则此列表中的所有元素必须实现{@link Comparable}接口，
     * 并且应使用元素'{@linkplain Comparable natural ordering}。
     *
     * 此列表必须是可修改的，但无需调整大小。
     *
     * @implSpec
     * 默认实现获取包含此列表中所有元素的数组，对数组进行排序，并迭代此列表，从数组中的相应
     * 位置重置每个元素。 （这样可以避免因尝试对链接列表进行排序而导致的n2 log（n）性能。）
     *
     * @implNote
     * 此实现是一个稳定的，自适应的迭代合并输出，当输入数组被部分排序时，它需要远远少于n 
     * lg（n）的比较，同时在输入数组随机排序时提供传统合并输出的性能。 如果输入数组几乎排
     * 序，则实现需要大约n次比较。 临时存储要求从几乎排序的输入数组的小常量到随机排序的输
     * 入数组的n / 2个对象引用不等。
     *
     * 该实现在其输入数组中具有升序和降序的相同优势，并且可以利用同一输入数组的不同部分中的
     * 升序和降序。 它非常适合合并两个或多个排序数组：只需连接数组并对结果数组进行排序。
     *
     * 该实现改编自Tim Peters的Python列表排序
     *（<a href="http://svn.python.org/projects/python/trunk/Objects/listsort.txt"> TimSort </a>）。
     * 它使用了Peter McIlroy的“乐观排序和信息理论复杂性”中的技术，参见“第四届年度ACM-SIAM离散算法研
     * 讨会论文集”，第467-474页，1993年1月。
     *
     * @param c the {@code Comparator} used to compare list elements.
     *          A {@code null} value indicates that the elements'
     *          {@linkplain Comparable natural ordering} should be used
     * @throws ClassCastException if the list contains elements that are not
     *         <i>mutually comparable</i> using the specified comparator
     * @throws UnsupportedOperationException if the list's list-iterator does
     *         not support the {@code set} operation
     * @throws IllegalArgumentException
     *         (<a href="Collection.html#optional-restrictions">optional</a>)
     *         if the comparator is found to violate the {@link Comparator}
     *         contract
     * @since 1.8
     */
    @SuppressWarnings({"unchecked", "rawtypes"})
    default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }

    /**
     * 从此列表中删除所有元素（可选操作）。 此调用返回后，列表将为空。
     *
     * @throws UnsupportedOperationException if the <tt>clear</tt> operation
     *         is not supported by this list
     */
    void clear();


    // Comparison and hashing

    /**
     * 将指定对象与此列表进行比较以获得相等性。 
     * 当且仅当指定的对象也是列表时，返回true，两个列表具有相同的大小，并且两个列表中的
     * 所有对应元素对都相等。 
     *（如果（e1 == null？e2 == null：e1.equals（e2）），则两个元素e1和e2相等。）
     * 换句话说，如果两个列表包含相同顺序的相同元素，则它们被定义为相等。 
     * 此定义确保equals方法在List接口的不同实现中正常工作。
     *
     * @param o the object to be compared for equality with this list
     * @return <tt>true</tt> if the specified object is equal to this list
     */
    boolean equals(Object o);

    /**
     * 返回此列表的哈希码值。 列表的哈希码被定义为以下计算的结果：
     * <pre>{@code
     *     int hashCode = 1;
     *     for (E e : list)
     *         hashCode = 31*hashCode + (e==null ? 0 : e.hashCode());
     * }</pre>
     * 这确保list1.equals（list2）暗示list1.hashCode（）== list2.hashCode（）对于
     * 任何两个列表list1和list2，如{@link Object＃hashCode}的常规合同所要求的。
     *
     * @return the hash code value for this list
     * @see Object#equals(Object)
     * @see #equals(Object)
     */
    int hashCode();


    // Positional Access Operations

    /**
     * 返回此列表中指定位置的元素。
     *
     * @param index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         (<tt>index &lt; 0 || index &gt;= size()</tt>)
     */
    E get(int index);

    /**
     * 用指定的元素替换此列表中指定位置的元素（可选操作）。
     *
     * @param index index of the element to replace
     * @param element element to be stored at the specified position
     * @return the element previously at the specified position
     * @throws UnsupportedOperationException if the <tt>set</tt> operation
     *         is not supported by this list
     * @throws ClassCastException if the class of the specified element
     *         prevents it from being added to this list
     * @throws NullPointerException if the specified element is null and
     *         this list does not permit null elements
     * @throws IllegalArgumentException if some property of the specified
     *         element prevents it from being added to this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         (<tt>index &lt; 0 || index &gt;= size()</tt>)
     */
    E set(int index, E element);

    /**
     * 将指定元素插入此列表中的指定位置（可选操作）。
     * 将当前位置的元素（如果有）和任何后续元素向右移动（将其添加到索引中）。
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws UnsupportedOperationException if the <tt>add</tt> operation
     *         is not supported by this list
     * @throws ClassCastException if the class of the specified element
     *         prevents it from being added to this list
     * @throws NullPointerException if the specified element is null and
     *         this list does not permit null elements
     * @throws IllegalArgumentException if some property of the specified
     *         element prevents it from being added to this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         (<tt>index &lt; 0 || index &gt; size()</tt>)
     */
    void add(int index, E element);

    /**
     * 删除此列表中指定位置的元素（可选操作）。 将任何后续元素向左移位（从索引
     * 中减去一个元素）。 返回从列表中删除的元素。
     *
     * @param index the index of the element to be removed
     * @return the element previously at the specified position
     * @throws UnsupportedOperationException if the <tt>remove</tt> operation
     *         is not supported by this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         (<tt>index &lt; 0 || index &gt;= size()</tt>)
     */
    E remove(int index);


    // Search Operations

    /**
     * 返回此列表中第一次出现的指定元素的索引，如果此列表不包含该元素，则返回-1。 更正
     * 式地，返回最低索引i，使得（o == null？get（i）== null：o.equals（get（i））），
     * 如果没有这样的索引则返回-1。
     *
     * @param o element to search for
     * @return the index of the first occurrence of the specified element in
     *         this list, or -1 if this list does not contain the element
     * @throws ClassCastException if the type of the specified element
     *         is incompatible with this list
     *         (<a href="Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified element is null and this
     *         list does not permit null elements
     *         (<a href="Collection.html#optional-restrictions">optional</a>)
     */
    int indexOf(Object o);

    /**
     * 返回此列表中指定元素最后一次出现的索引，如果此列表不包含该元素，则返回-1。 更
     * 正式地，返回最高索引i，使得（o == null？get（i）== null：o.equals（get（i））），
     * 如果没有这样的索引则返回-1。
     *
     * @param o element to search for
     * @return the index of the last occurrence of the specified element in
     *         this list, or -1 if this list does not contain the element
     * @throws ClassCastException if the type of the specified element
     *         is incompatible with this list
     *         (<a href="Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if the specified element is null and this
     *         list does not permit null elements
     *         (<a href="Collection.html#optional-restrictions">optional</a>)
     */
    int lastIndexOf(Object o);


    // List Iterators

    /**
     * 返回此列表中元素的列表迭代器（按适当顺序）。
     *
     * @return a list iterator over the elements in this list (in proper
     *         sequence)
     */
    ListIterator<E> listIterator();

    /**
     * 从列表中的指定位置开始，返回列表中元素的列表迭代器（按正确顺序）。 指定的索
     * 引表示初始调用{@link ListIterator＃next next}将返回的第一个元素。 对
     * {@link ListIterator＃previous previous}的初始调用将返回指定索引减去1的元素。
     *
     * @param index index of the first element to be returned from the
     *        list iterator (by a call to {@link ListIterator#next next})
     * @return a list iterator over the elements in this list (in proper
     *         sequence), starting at the specified position in the list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index > size()})
     */
    ListIterator<E> listIterator(int index);

    // View

    /**
     * 返回指定fromIndex（包含）和toIndex（独占）之间此列表部分的视图。 （如
     * 果fromIndex和toIndex相等，则返回的列表为空。）返回的列表由此列表支持，
     * 因此返回列表中的非结构更改将反映在此列表中，反之亦然。 返回的列表支持
     * 此列表支持的所有可选列表操作。
     *
     *  此方法消除了对显式范围操作的需要（对于数组通常存在的排序）。 
     * 任何需要列表的操作都可以通过传递subList视图而不是整个列表来用作范围操作。
     * 例如，以下习语从列表中删除了一系列元素：
     * <pre>{@code
     *      list.subList(from, to).clear();
     * }</pre>
     * 可以为indexOf和lastIndexOf构造类似的习语，并且可以将Collections类中的所有
     * 算法应用于subList。
     *
     * 如果支持列表（即，此列表）在结构上以除了返回列表之外的任何方式进行修改，则
     * 此方法返回的列表的语义将变为未定义。 
     *（结构修改是那些改变了这个列表的大小，或以其他方式扰乱它的方式，正在进行的迭
     * 代可能会产生不正确的结果。）
     *
     * @param fromIndex low endpoint (inclusive) of the subList
     * @param toIndex high endpoint (exclusive) of the subList
     * @return a view of the specified range within this list
     * @throws IndexOutOfBoundsException for an illegal endpoint index value
     *         (<tt>fromIndex &lt; 0 || toIndex &gt; size ||
     *         fromIndex &gt; toIndex</tt>)
     */
    List<E> subList(int fromIndex, int toIndex);

    /**
     * 在此列表中的元素上创建{@link Spliterator}。
     *
     * {@code Spliterator}报告{@link Spliterator＃SIZED}和{@link Spliterator＃ORDERED}。 
     * 实施应记录其他特征值的报告。
     *
     * @implSpec
     * 默认实现从列表的{@code Iterator}创建<a href="Spliterator.html#binding">后期绑定
     * </a>分裂器。 spliterator继承了列表迭代器的fail-fast属性。
     *
     * @implNote
     * 创建的{@code Spliterator}还会报告{@link Spliterator＃SUBSIZED}。
     *
     * @return a {@code Spliterator} over the elements in this list
     * @since 1.8
     */
    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, Spliterator.ORDERED);
    }
}

```