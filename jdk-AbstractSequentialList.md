```java
package java.util;

/**
 * 这个类提供了List接口的框架实现，以最小化实现“顺序访问”数据存储（如链接列表）支
 * 持的此接口所需的工作。对于随机访问数据（例如数组），应该优先使用AbstractList。
 *
 * 这个类与AbstractList类相反，因为它在列表迭代器的顶部实现“随机访问”方法
 * （get（int index）、set（int index，E元素）、add（int index，E元素）和.（int index）），
 * 而不是相反。
 *
 * 
 * 为了实现一个列表，程序员只需要扩展这个类，并为listIterator和大小方法提供实现。对于不可
 * 修改的列表，程序员只需实现列表迭代器的hasNext、next、hasPrevious、forefore和index方法。
 *
 * 对于可修改的列表，程序员应该另外实现列表迭代器的set方法。对于大小可变的列表，程序员应该
 * 另外实现列表迭代器的remove和add方法。
 *
 * 根据Collection接口规范中的建议，程序员通常应该提供一个void（无参数）和集合构造函数。
 *
 * This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @author  Josh Bloch
 * @author  Neal Gafter
 * @see Collection
 * @see List
 * @see AbstractList
 * @see AbstractCollection
 * @since 1.2
 */

public abstract class AbstractSequentialList<E> extends AbstractList<E> {
    /**
     * 独立构造函数.（对于子类构造函数的调用，通常是隐式的。）
     */
    protected AbstractSequentialList() {
    }

    /**
     * 返回列表中指定位置的元素。
     *
     * 这个实现首先得到一个指向索引元素的列表迭代器（带有listIterator（index））。
     * 然后，它使用ListIterator.next获取元素并返回它。
     *
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E get(int index) {
        try {
            return listIterator(index).next();
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }

    /**
     * 用指定的元素替换列表中指定位置的元素（可选操作）。
     *
     * 这个实现首先得到一个指向索引元素的列表迭代器（带有listIterator（index））。然后，它使用
     * ListIterator.next获取当前元素，并将其替换为ListIterator.set。
     *
     * 注意，如果列表迭代器未实现设置操作，则此实现将引发UnsupportedOperationException。
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     * @throws IllegalArgumentException      {@inheritDoc}
     * @throws IndexOutOfBoundsException     {@inheritDoc}
     */
    public E set(int index, E element) {
        try {
            ListIterator<E> e = listIterator(index);
            E oldVal = e.next();
            e.set(element);
            return oldVal;
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }

    /**
     * 在此列表中的指定位置插入指定元素（可选操作）。将当前位于该位置的元素（如果有的话）和
     * 任何后续元素向右移动（向它们的索引中添加一个）。
     *
     * 这个实现首先得到一个指向索引元素的列表迭代器（带有listIterator（index））。然后，它用
     * ListIterator.add插入指定的元素。
     *
     * 注意，如果列表迭代器没有实现添加操作，则此实现将抛出UnsupportedOperationException。
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     * @throws IllegalArgumentException      {@inheritDoc}
     * @throws IndexOutOfBoundsException     {@inheritDoc}
     */
    public void add(int index, E element) {
        try {
            listIterator(index).add(element);
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }

    /**
     * 删除列表中指定位置的元素（可选操作）。向左移动任何后续元素（从其索引中减去一个）。返回从列
     * 表中删除的元素。
     *
     * 这个实现首先得到一个指向索引元素的列表迭代器（带有listIterator（index））。然后，它用
     * ListIterator.remove删除元素。
     *
     * 注意，如果列表迭代器没有实现删除操作，则此实现将抛出UnsupportedOperationException。
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws IndexOutOfBoundsException     {@inheritDoc}
     */
    public E remove(int index) {
        try {
            ListIterator<E> e = listIterator(index);
            E outCast = e.next();
            e.remove();
            return outCast;
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }


    // Bulk Operations

    /**
     * 将指定集合中的所有元素插入到指定位置的列表中（可选操作）。将当前位于该位置的元素
     *（如果有的话）和任何后续元素向右移动（增加它们的索引）。新元素将以指定集合的迭代器
     * 返回的顺序出现在此列表中。如果在操作进行中修改指定的集合，则此操作的行为未定义。
     *（注意，如果指定的集合是这个列表，并且它是非空的，则会发生这种情况。）
     *
     * 此实现获取指定集合上的迭代器和指向索引元素的列表迭代器（使用listIterator（index））。
     * 然后，它在指定的集合上迭代，使用ListIterator.add后面跟着ListIterator.next（跳过添加
     * 的元素），将从迭代器获得的元素一次一个地插入到该列表中。
     *
     * 注意，如果listIterator方法返回的列表迭代器没有实现添加操作，则此实现将抛出
     * UnsupportedOperationException。
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     * @throws IllegalArgumentException      {@inheritDoc}
     * @throws IndexOutOfBoundsException     {@inheritDoc}
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        try {
            boolean modified = false;
            ListIterator<E> e1 = listIterator(index);
            Iterator<? extends E> e2 = c.iterator();
            while (e2.hasNext()) {
                e1.add(e2.next());
                modified = true;
            }
            return modified;
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }


    // Iterators

    /**
     * 返回对该列表中的元素的迭代器（按适当的顺序）。
     *
     * 这个实现只返回列表上的列表迭代器。
     *
     * @return an iterator over the elements in this list (in proper sequence)
     */
    public Iterator<E> iterator() {
        return listIterator();
    }

    /**
     * 返回此列表中的元素的列表迭代器（按适当顺序）。
     *
     * @param  index index of first element to be returned from the list
     *         iterator (by a call to the <code>next</code> method)
     * @return a list iterator over the elements in this list (in proper
     *         sequence)
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public abstract ListIterator<E> listIterator(int index);
}

```