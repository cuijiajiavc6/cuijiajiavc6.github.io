```java
package java.util;

/**
 * 此类提供{@link List}接口的骨干实现，以最大限度地减少实现由“随机访问”数据存储
 * （例如数组）支持的此接口所需的工作量。 对于顺序访问数据（例如链接列表），应优
 * 先使用{@link AbstractSequentialList}作为此类。
 *
 * 要实现不可修改的列表，程序员只需要扩展此类并提供{@link #get（int）}和
 * {@link List＃size（）size（）}方法的实现。
 *
 * 要实现可修改的列表，程序员必须另外覆盖{@link #set（int，Object）set（int，E）}方
 * 法（否则会抛出{@code UnsupportedOperationException}）。 如果列表是可变大小的，则
 * 程序员必须另外覆盖{@link #add（int，Object）add（int，E）}和{@link #remove（int）}方法。
 *
 * 程序员通常应该根据{@link Collection}接口规范中的建议提供void（无参数）和集合构造函数。
 *
 * 与其他抽象集合实现不同，程序员不必提供迭代器实现; 迭代器和列表迭代器是由这个类在“随机访
 * 问”方法之上实现的：
 * {@link #get(int)},
 * {@link #set(int, Object) set(int, E)},
 * {@link #add(int, Object) add(int, E)} and
 * {@link #remove(int)}.
 *
 * 此类中每个非抽象方法的文档详细描述了它的实现。 如果正在实施的集合允许更有效的实现，则可
 * 以覆盖这些方法中的每一个。
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @author  Josh Bloch
 * @author  Neal Gafter
 * @since 1.2
 */

public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
    /**
     * Sole constructor.  (For invocation by subclass constructors, typically
     * implicit.)
     */
    protected AbstractList() {
    }

    /**
     * 将指定的元素追加到此列表的末尾（可选操作）。
     *
     * 支持此操作的列表可能会限制可能添加到此列表的元素。
     * 特别是，某些列表将拒绝添加null元素，而其他列表将对可能添加的元素类型施加限制。 
     * 列表类应在其文档中明确指出可以添加哪些元素的任何限制。
     *
     * <p>这实现调用 {@code add(size(), e)}.
     *
     * 请注意，除非重写{@link #add（int，Object）add（int，E）}，否则此实现会抛出
     * {@code UnsupportedOperationException}。
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     * @throws UnsupportedOperationException if the {@code add} operation
     *         is not supported by this list
     * @throws ClassCastException if the class of the specified element
     *         prevents it from being added to this list
     * @throws NullPointerException if the specified element is null and this
     *         list does not permit null elements
     * @throws IllegalArgumentException if some property of this element
     *         prevents it from being added to this list
     */
    public boolean add(E e) {
        add(size(), e);
        return true;
    }

    /**
     * {@inheritDoc}
     *
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    abstract public E get(int index);

    /**
     * {@inheritDoc}
     *
     * <p>This implementation always throws an
     * {@code UnsupportedOperationException}.
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     * @throws IllegalArgumentException      {@inheritDoc}
     * @throws IndexOutOfBoundsException     {@inheritDoc}
     */
    public E set(int index, E element) {
        throw new UnsupportedOperationException();
    }

    /**
     * {@inheritDoc}
     *
     * <p>This implementation always throws an
     * {@code UnsupportedOperationException}.
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     * @throws IllegalArgumentException      {@inheritDoc}
     * @throws IndexOutOfBoundsException     {@inheritDoc}
     */
    public void add(int index, E element) {
        throw new UnsupportedOperationException();
    }

    /**
     * {@inheritDoc}
     *
     * <p>This implementation always throws an
     * {@code UnsupportedOperationException}.
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws IndexOutOfBoundsException     {@inheritDoc}
     */
    public E remove(int index) {
        throw new UnsupportedOperationException();
    }


    // Search Operations

    /**
     * {@inheritDoc}
     *
     * 此实现首先获取列表迭代器（使用{@code listIterator（）}）。 然后，它遍
     * 历列表，直到找到指定的元素或到达列表的末尾。
     *
     * @throws ClassCastException   {@inheritDoc}
     * @throws NullPointerException {@inheritDoc}
     */
    public int indexOf(Object o) {
        ListIterator<E> it = listIterator();
        if (o==null) {
            while (it.hasNext())
                if (it.next()==null)
                    return it.previousIndex();
        } else {
            while (it.hasNext())
                if (o.equals(it.next()))
                    return it.previousIndex();
        }
        return -1;
    }

    /**
     * {@inheritDoc}
     *
     * 此实现首先获取指向列表末尾的列表迭代器（使用{@code listIterator（size（））}）。 
     * 然后，它在列表上向后迭代，直到找到指定的元素，或者到达列表的开头。
     *
     * @throws ClassCastException   {@inheritDoc}
     * @throws NullPointerException {@inheritDoc}
     */
    public int lastIndexOf(Object o) {
        ListIterator<E> it = listIterator(size());
        if (o==null) {
            while (it.hasPrevious())
                if (it.previous()==null)
                    return it.nextIndex();
        } else {
            while (it.hasPrevious())
                if (o.equals(it.previous()))
                    return it.nextIndex();
        }
        return -1;
    }


    // Bulk Operations

    /**
     * 从此列表中删除所有元素（可选操作）。 此调用返回后，列表将为空。
     *
     * <p>This implementation calls {@code removeRange(0, size())}.
     *
     * 请注意，除非{@code remove（int index）}或{@code removeRange
     *（int fromIndex，int toIndex）}被覆盖，否则此实现会抛出
     * {@code UnsupportedOperationException}。
     *
     * @throws UnsupportedOperationException if the {@code clear} operation
     *         is not supported by this list
     */
    public void clear() {
        removeRange(0, size());
    }

    /**
     * {@inheritDoc}
     *
     * 此实现获取指定集合上的迭代器并对其进行迭代，使用{@code 
     * add（int，E）}将从迭代器获取的元素一次一个地插入到此列表中的适当位置。
     * 许多实现将覆盖此方法以提高效率。
     *
     * 请注意，除非重写{@link #add（int，Object）add（int，E）}，否则此实现会
     * 出{@code UnsupportedOperationException}。
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     * @throws IllegalArgumentException      {@inheritDoc}
     * @throws IndexOutOfBoundsException     {@inheritDoc}
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);
        boolean modified = false;
        for (E e : c) {
            add(index++, e);
            modified = true;
        }
        return modified;
    }


    // Iterators

    /**
     * 以适当的顺序返回此列表中元素的迭代器。
     *
     * 此实现返回迭代器接口的简单实现，依赖于支持列表的{@code size（）}，
     * {@ code get（int）}和{@code remove（int）}方法。
     *
     * 请注意，此方法返回的迭代器将抛出{@link UnsupportedOperationException}
     * 以响应其{@code remove}方法，除非重写了列表的{@code remove（int）}方法。
     *
     * 可以使该实现在并发修改时抛出运行时异常，如（受保护的）{@link #modCount}
     * 字段的规范中所述。
     *
     * @return an iterator over the elements in this list in proper sequence
     */
    public Iterator<E> iterator() {
        return new Itr();
    }

    /**
     * {@inheritDoc}
     *
     * <p>This implementation returns {@code listIterator(0)}.
     *
     * @see #listIterator(int)
     */
    public ListIterator<E> listIterator() {
        return listIterator(0);
    }

    /**
     * {@inheritDoc}
     *
     * 此实现返回{@code ListIterator}接口的简单实现，该接口扩展了{@code iterator（）}方
     * 法返回的{@code Iterator}接口的实现。 {@code ListIterator}实现依赖于支持列表的
     * {@code get（int）}，{@ code set（int，E）}，{@ code add（int，E）}和
     * {@code remove（int） } 方法。
     *
     * 请注意，此实现返回的列表迭代器将抛出{@link UnsupportedOperationException}以响应
     * 其{@code remove}，{@ code set}和{@code add}方法，除非列表的{@code remove（int）} ，
     * {@code set（int，E）}和{@code add（int，E）}方法被覆盖。
     *
     * 可以使该实现在并发修改时抛出运行时异常，如（受保护的）{@link #modCount}字段的规范
     * 中所述。
     *
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public ListIterator<E> listIterator(final int index) {
        rangeCheckForAdd(index);

        return new ListItr(index);
    }

    private class Itr implements Iterator<E> {
        /**
         * 后续调用next返回的元素索引。
         */
        int cursor = 0;

        /**
         * 最近一次调用返回到下一个或上一个的元素索引。 如果通过删除调用删除此元素，则重置为-1。
         */
        int lastRet = -1;

        /**
         * 迭代器认为后备列表应具有的modCount值。 如果违反了此期望，则迭代器已检测到并发修改。
         */
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size();
        }

        public E next() {
            checkForComodification();
            try {
                int i = cursor;
                E next = get(i);
                lastRet = i;
                cursor = i + 1;
                return next;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                AbstractList.this.remove(lastRet);
                if (lastRet < cursor)
                    cursor--;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException e) {
                throw new ConcurrentModificationException();
            }
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

    private class ListItr extends Itr implements ListIterator<E> {
        ListItr(int index) {
            cursor = index;
        }

        public boolean hasPrevious() {
            return cursor != 0;
        }

        public E previous() {
            checkForComodification();
            try {
                int i = cursor - 1;
                E previous = get(i);
                lastRet = cursor = i;
                return previous;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        public int nextIndex() {
            return cursor;
        }

        public int previousIndex() {
            return cursor-1;
        }

        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                AbstractList.this.set(lastRet, e);
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                AbstractList.this.add(i, e);
                lastRet = -1;
                cursor = i + 1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }

    /**
     * {@inheritDoc}
     *
     * 此实现返回一个子类{@code AbstractList}的列表。 
     * 子类在私有字段中存储后备列表中subList的偏移量，subList的大小（可以在其生
     * 命周期内更改），以及后备列表的预期{@code modCount}值。 子类有两种变体，
     * 其中一种实现{@code RandomAccess}。 如果此列表实现{@code RandomAccess}，则返
     * 回的列表将是实现{@code RandomAccess}的子类的实例。
     *
     * 子类的{@code set（int，E）}，{@ code get（int）}，{@ code add（int，E）}，
     * {@ code remove（int）}，{@ code addAll（int，Collection }}和{@code 
     * removeRange（int，int）}方法在绑定检查索引并调整偏移量之后，全部委托给后备抽象
     * 列表上的相应方法。 {@code addAll（Collection c）}方法仅返回{@code addAll（size，c）}。
     *
     * {@code listIterator（int）}方法在支持列表上的列表迭代器上返回“包装器对象”，该列表
     * 迭代器是使用支持列表上的相应方法创建的。 {@code iterator}方法只返回
     * {@code listIterator（）}，而{@code size}方法只返回子类的{@code size}字段。
     *
     * 所有方法首先检查后备列表的实际{@code modCount}是否等于其预期值，如果不是，则抛
     * 出{@code ConcurrentModificationException}。
     *
     * @throws IndexOutOfBoundsException if an endpoint index value is out of range
     *         {@code (fromIndex < 0 || toIndex > size)}
     * @throws IllegalArgumentException if the endpoint indices are out of order
     *         {@code (fromIndex > toIndex)}
     */
    public List<E> subList(int fromIndex, int toIndex) {
        return (this instanceof RandomAccess ?
                new RandomAccessSubList<>(this, fromIndex, toIndex) :
                new SubList<>(this, fromIndex, toIndex));
    }

    // Comparison and hashing

    /**
     * 将指定对象与此列表进行比较以获得相等性。 当且仅当指定的对象也是列表时，返回{@code 
     * true}，两个列表具有相同的大小，并且两个列表中的所有对应元素对都是<i>相等</ i>。 
     *（如果{@code（e1 == null？e2 == null：e1.equals（e2））}，两个元素{@code e1}和
     * {@code e2} <i>相等</ i>。）其他 单词，如果两个列表包含相同顺序的相同元素，则它们
     * 被定义为相等。
     *
     * 此实现首先检查指定的对象是否为此列表。 如果是，则返回{@code true}; 如果不是，它检查
     * 指定的对象是否是列表。 如果没有，则返回{@code false}; 如果是这样，它迭代两个列表，
     * 比较相应的元素对。 如果任何比较返回{@code false}，则此方法返回{@code false}。 
     * 如果迭代器在另一个之前耗尽元素，则返回{@code false}（因为列表的长度不等）; 否则它会
     * 在迭代完成时返回{@code true}。
     *
     * @param o the object to be compared for equality with this list
     * @return {@code true} if the specified object is equal to this list
     */
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof List))
            return false;

        ListIterator<E> e1 = listIterator();
        ListIterator<?> e2 = ((List<?>) o).listIterator();
        while (e1.hasNext() && e2.hasNext()) {
            E o1 = e1.next();
            Object o2 = e2.next();
            if (!(o1==null ? o2==null : o1.equals(o2)))
                return false;
        }
        return !(e1.hasNext() || e2.hasNext());
    }

    /**
     * 返回此列表的哈希码值。
     *
     * 此实现完全使用用于在{@link List＃hashCode}方法的文档中定义列表哈希函数的代码。
     *
     * @return the hash code value for this list
     */
    public int hashCode() {
        int hashCode = 1;
        for (E e : this)
            hashCode = 31*hashCode + (e==null ? 0 : e.hashCode());
        return hashCode;
    }

    /**
     * 从此列表中删除索引在{@code fromIndex}之间的所有元素，包括{@code toIndex}，独占。
     * 将任何后续元素向左移动（降低其索引）。 此调用通过{@code（toIndex  -  fromIndex）}元
     * 素缩短列表。 （如果{@code toIndex == fromIndex}，此操作无效。）
     *
     * 此方法由此列表及其子列表上的{@code clear}操作调用。 
     * 重写此方法以利用列表实现的内部可以显着提高此列表及其子列表上的{@code clear}操作的性能。
     *
     * 此实现在{@code fromIndex}之前获取一个列表迭代器，并重复调用{@code ListIterator.next}，
     * 然后调用{@code ListIterator.remove}，直到整个范围被删除。 注意：如果
     * {@code ListIterator.remove}需要线性时间，则此实现需要二次时间。     
     *  
     * @param fromIndex index of first element to be removed
     * @param toIndex index after last element to be removed
     */
    protected void removeRange(int fromIndex, int toIndex) {
        ListIterator<E> it = listIterator(fromIndex);
        for (int i=0, n=toIndex-fromIndex; i<n; i++) {
            it.next();
            it.remove();
        }
    }

    /**
     * 此列表已被结构修改的次数。 
     * 结构修改是那些改变列表大小或以其他方式扰乱它的方式，即正在进行的迭
     * 代可能产生不正确的结果。
     *
     * 该字段由{@code iterator}和{@code listIterator}方法返回的迭代器和列表迭代器实现使用。
     * 如果此字段的值意外更改，则迭代器（或列表迭代器）将抛出
     * {@code ConcurrentModificationException}以响应{@code next}，{@ code remove}，
     * {@ code previous}，{@ code set }或{@code add}操作。 
     * 这提供了快速失败的行为，而不是在迭代期间面对并发修改时的非确定性行为。
     *
     * 子类对此字段的使用是可选的。 如果子类希望提供快速失败的迭代器（并列出迭代器），那么
     * 它只需要在其{@code add（int，E）}和{@code remove（int）}方法（以及任何其他方法）中增
     * 加此字段 它覆盖的方法导致对列表的结构修改）。 对{@code add（int，E）}或
     * {@code remove（int）}的单个调用必须向该字段添加不超过一个，否则迭代器（和列表迭代器）\
     * 将抛出伪造的{@code ConcurrentModificationExceptions}。 如果实现不希望提供快速失败的迭代
     * 器，则可以忽略此字段。
     * 
     */
    protected transient int modCount = 0;

    private void rangeCheckForAdd(int index) {
        if (index < 0 || index > size())
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size();
    }
}

class SubList<E> extends AbstractList<E> {
    private final AbstractList<E> l;
    private final int offset;
    private int size;

    SubList(AbstractList<E> list, int fromIndex, int toIndex) {
        if (fromIndex < 0)
            throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
        if (toIndex > list.size())
            throw new IndexOutOfBoundsException("toIndex = " + toIndex);
        if (fromIndex > toIndex)
            throw new IllegalArgumentException("fromIndex(" + fromIndex +
                                               ") > toIndex(" + toIndex + ")");
        l = list;
        offset = fromIndex;
        size = toIndex - fromIndex;
        this.modCount = l.modCount;
    }

    public E set(int index, E element) {
        rangeCheck(index);
        checkForComodification();
        return l.set(index+offset, element);
    }

    public E get(int index) {
        rangeCheck(index);
        checkForComodification();
        return l.get(index+offset);
    }

    public int size() {
        checkForComodification();
        return size;
    }

    public void add(int index, E element) {
        rangeCheckForAdd(index);
        checkForComodification();
        l.add(index+offset, element);
        this.modCount = l.modCount;
        size++;
    }

    public E remove(int index) {
        rangeCheck(index);
        checkForComodification();
        E result = l.remove(index+offset);
        this.modCount = l.modCount;
        size--;
        return result;
    }

    protected void removeRange(int fromIndex, int toIndex) {
        checkForComodification();
        l.removeRange(fromIndex+offset, toIndex+offset);
        this.modCount = l.modCount;
        size -= (toIndex-fromIndex);
    }

    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }

    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);
        int cSize = c.size();
        if (cSize==0)
            return false;

        checkForComodification();
        l.addAll(offset+index, c);
        this.modCount = l.modCount;
        size += cSize;
        return true;
    }

    public Iterator<E> iterator() {
        return listIterator();
    }

    public ListIterator<E> listIterator(final int index) {
        checkForComodification();
        rangeCheckForAdd(index);

        return new ListIterator<E>() {
            private final ListIterator<E> i = l.listIterator(index+offset);

            public boolean hasNext() {
                return nextIndex() < size;
            }

            public E next() {
                if (hasNext())
                    return i.next();
                else
                    throw new NoSuchElementException();
            }

            public boolean hasPrevious() {
                return previousIndex() >= 0;
            }

            public E previous() {
                if (hasPrevious())
                    return i.previous();
                else
                    throw new NoSuchElementException();
            }

            public int nextIndex() {
                return i.nextIndex() - offset;
            }

            public int previousIndex() {
                return i.previousIndex() - offset;
            }

            public void remove() {
                i.remove();
                SubList.this.modCount = l.modCount;
                size--;
            }

            public void set(E e) {
                i.set(e);
            }

            public void add(E e) {
                i.add(e);
                SubList.this.modCount = l.modCount;
                size++;
            }
        };
    }

    public List<E> subList(int fromIndex, int toIndex) {
        return new SubList<>(this, fromIndex, toIndex);
    }

    private void rangeCheck(int index) {
        if (index < 0 || index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private void rangeCheckForAdd(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    private void checkForComodification() {
        if (this.modCount != l.modCount)
            throw new ConcurrentModificationException();
    }
}

class RandomAccessSubList<E> extends SubList<E> implements RandomAccess {
    RandomAccessSubList(AbstractList<E> list, int fromIndex, int toIndex) {
        super(list, fromIndex, toIndex);
    }

    public List<E> subList(int fromIndex, int toIndex) {
        return new RandomAccessSubList<>(this, fromIndex, toIndex);
    }
}

```