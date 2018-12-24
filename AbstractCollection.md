```java
package java.util;

/**
 * 此类提供Collection接口的骨干实现，以最大限度地减少实现此接口所需的工作量。
 *
 * 要实现不可修改的集合，程序员只需要扩展此类并提供迭代器和size方法的实现。 
 *（迭代器方法返回的迭代器必须实现hasNext和next。）
 *
 * 要实现可修改的集合，程序员必须另外覆盖此类的add方法（否则会抛出
 * UnsupportedOperationException），迭代器方法返回的迭代器必须另外实现其remove
 * 方法。
 *
 * 程序员通常应该根据Collection接口规范中的建议提供void（无参数）和Collection
 * 构造函数。
 *
 * 此类中每个非抽象方法的文档详细描述了它的实现。 如果正在实施的集合允许更有效
 * 的实现，则可以覆盖这些方法中的每一个。
 *
 * This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @author  Josh Bloch
 * @author  Neal Gafter
 * @see Collection
 * @since 1.2
 */

public abstract class AbstractCollection<E> implements Collection<E> {
    /**
     * 唯一的构造函数。 （对于子类构造函数的调用，通常是隐式的。）
     */
    protected AbstractCollection() {
    }

    // Query Operations

    /**
     * 返回此collection中包含的元素的迭代器。
     *
     * @return an iterator over the elements contained in this collection
     */
    public abstract Iterator<E> iterator();

    public abstract int size();

    /**
     * {@inheritDoc}
     *
     * <p>This implementation returns <tt>size() == 0</tt>.
     */
    public boolean isEmpty() {
        return size() == 0;
    }

    /**
     * {@inheritDoc}
     *
     * 此实现迭代集合中的元素，依次检查每个元素是否与指定元素相等。
     *
     * @throws ClassCastException   {@inheritDoc}
     * @throws NullPointerException {@inheritDoc}
     */
    public boolean contains(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext())
                if (it.next()==null)
                    return true;
        } else {
            while (it.hasNext())
                if (o.equals(it.next()))
                    return true;
        }
        return false;
    }

    /**
     * {@inheritDoc}
     *
     * 此实现返回一个数组，该数组包含此集合的迭代器返回的所有元素，以相同的顺序存储
     * 在数组的连续元素中，从索引{@code 0}开始。 返回数组的长度等于迭代器返回的元素
     * 数，即使此集合的大小在迭代期间发生更改，如果集合允许在迭代期间进行并发修改，
     * 也可能发生这种情况。 {@code size}方法仅作为优化提示调用; 即使迭代器返回不同数
     * 量的元素，也会返回正确的结果。
     *
     * 这个方法等同于下面的代码:
     *
     *  <pre> {@code
     * List<E> list = new ArrayList<E>(size());
     * for (E e : this)
     *     list.add(e);
     * return list.toArray();
     * }</pre>
     */
    public Object[] toArray() {
        // Estimate size of array; be prepared to see more or fewer elements
        Object[] r = new Object[size()];
        Iterator<E> it = iterator();
        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) // fewer elements than expected
                return Arrays.copyOf(r, i);
            r[i] = it.next();
        }
        return it.hasNext() ? finishToArray(r, it) : r;
    }

    /**
     * {@inheritDoc}
     *
     * 此实现返回一个数组，该数组包含此集合的迭代器以相同顺序返回的所有元素，
     * 存储在数组的连续元素中，从索引{@code 0}开始。 如果迭代器返回的元素数
     * 量太大而不适合指定的数组，那么元素将在新分配的数组中返回，其长度等于
     * 迭代器返回的元素数，即使此集合的大小发生更改 在迭代期间，如果集合允许
     * 在迭代期间进行并发修改，则可能发生这种情况。 {@code size}方法仅作为优
     * 化提示调用; 即使迭代器返回不同数量的元素，也会返回正确的结果。
     *
     * 这个方法等同于下面的代码:
     *
     *  <pre> {@code
     * List<E> list = new ArrayList<E>(size());
     * for (E e : this)
     *     list.add(e);
     * return list.toArray(a);
     * }</pre>
     *
     * @throws ArrayStoreException  {@inheritDoc}
     * @throws NullPointerException {@inheritDoc}
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        // Estimate size of array; be prepared to see more or fewer elements
        int size = size();
        T[] r = a.length >= size ? a :
                  (T[])java.lang.reflect.Array
                  .newInstance(a.getClass().getComponentType(), size);
        Iterator<E> it = iterator();

        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) { // fewer elements than expected
                if (a == r) {
                    r[i] = null; // null-terminate
                } else if (a.length < i) {
                    return Arrays.copyOf(r, i);
                } else {
                    System.arraycopy(r, 0, a, 0, i);
                    if (a.length > i) {
                        a[i] = null;
                    }
                }
                return a;
            }
            r[i] = (T)it.next();
        }
        // more elements than expected
        return it.hasNext() ? finishToArray(r, it) : r;
    }

    /**
     * 要分配的最大数组大小。 一些VM在阵列中保留一些标题字。 
     * 尝试分配更大的数组可能会导致OutOfMemoryError：请求的数组大小超过VM限制
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * 当迭代器返回的元素多于预期时，重新分配在toArray中使用的数组，并完成从迭代器填充它。
     *
     * @param r the array, replete with previously stored elements
     * @param it the in-progress iterator over this collection
     * @return array containing the elements in the given array, plus any
     *         further elements returned by the iterator, trimmed to size
     */
    @SuppressWarnings("unchecked")
    private static <T> T[] finishToArray(T[] r, Iterator<?> it) {
        int i = r.length;
        while (it.hasNext()) {
            int cap = r.length;
            if (i == cap) {
                int newCap = cap + (cap >> 1) + 1;
                // overflow-conscious code
                if (newCap - MAX_ARRAY_SIZE > 0)
                    newCap = hugeCapacity(cap + 1);
                r = Arrays.copyOf(r, newCap);
            }
            r[i++] = (T)it.next();
        }
        // trim if overallocated
        return (i == r.length) ? r : Arrays.copyOf(r, i);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError
                ("Required array size too large");
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    // Modification Operations

    /**
     * {@inheritDoc}
     *
     * 此实现始终抛出UnsupportedOperationException。
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     * @throws IllegalArgumentException      {@inheritDoc}
     * @throws IllegalStateException         {@inheritDoc}
     */
    public boolean add(E e) {
        throw new UnsupportedOperationException();
    }

    /**
     * {@inheritDoc}
     *
     * 此实现迭代集合以查找指定的元素。 如果找到该元素，它将使用迭代器的remove方
     * 法从集合中删除该元素。
     *
     * 请注意，如果此集合的迭代器方法返回的迭代器未实现remove方法且此集合包含指定的对
     * 象，则此实现将抛出UnsupportedOperationException。
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     */
    public boolean remove(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext()) {
                if (it.next()==null) {
                    it.remove();
                    return true;
                }
            }
        } else {
            while (it.hasNext()) {
                if (o.equals(it.next())) {
                    it.remove();
                    return true;
                }
            }
        }
        return false;
    }


    // Bulk Operations

    /**
     * {@inheritDoc}
     *
     * 此实现迭代指定的集合，依次检查迭代器返回的每个元素以查看它是否包含在此集合中。 
     * 如果包含所有元素，则返回true，否则返回false。
     *
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     * @see #contains(Object)
     */
    public boolean containsAll(Collection<?> c) {
        for (Object e : c)
            if (!contains(e))
                return false;
        return true;
    }

    /**
     * {@inheritDoc}
     *
     * 此实现迭代指定的集合，并依次将迭代器返回的每个对象添加到此集合。
     *
     * 请注意，除非重写add（假设指定的collection非空），否则此实现将抛出
     * UnsupportedOperationException。
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     * @throws IllegalArgumentException      {@inheritDoc}
     * @throws IllegalStateException         {@inheritDoc}
     *
     * @see #add(Object)
     */
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }

    /**
     * {@inheritDoc}
     *
     * 此实现迭代此集合，依次检查迭代器返回的每个元素，以查看它是否包含在指定的集合中。 
     * 如果包含它，则使用迭代器的remove方法将其从此集合中删除。
     *
     * 请注意，如果迭代器方法返回的迭代器未实现remove方法，并且此collection包含与指定
     * collection相同的一个或多个元素，则此实现将抛出UnsupportedOperationException。
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     *
     * @see #remove(Object)
     * @see #contains(Object)
     */
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        boolean modified = false;
        Iterator<?> it = iterator();
        while (it.hasNext()) {
            if (c.contains(it.next())) {
                it.remove();
                modified = true;
            }
        }
        return modified;
    }

    /**
     * {@inheritDoc}
     *
     * 此实现迭代此集合，依次检查迭代器返回的每个元素，以查看它是否包含在指定的集合中。 
     * 如果它没有包含，则使用迭代器的remove方法将其从此集合中删除。
     *
     * 请注意，如果迭代器方法返回的迭代器未实现remove方法，并且此collection包含指定
     * collection中不存在的一个或多个元素，则此实现将抛出UnsupportedOperationException。
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     *
     * @see #remove(Object)
     * @see #contains(Object)
     */
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        boolean modified = false;
        Iterator<E> it = iterator();
        while (it.hasNext()) {
            if (!c.contains(it.next())) {
                it.remove();
                modified = true;
            }
        }
        return modified;
    }

    /**
     * {@inheritDoc}
     *
     * 此实现迭代此集合，使用Iterator.remove操作删除每个元素。 大多数实现可能会选
     * 择覆盖此方法以提高效率。
     *
     * 请注意，如果此集合的迭代器方法返回的迭代器未实现remove方法且此集合非空，则此实现将
     * 抛出UnsupportedOperationException。
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     */
    public void clear() {
        Iterator<E> it = iterator();
        while (it.hasNext()) {
            it.next();
            it.remove();
        }
    }


    //  String conversion

    /**
     * 返回此集合的字符串表示形式。 字符串表示由一个集合元素的列表组成，它们按迭代器返
     * 回的顺序排列，用方括号（“[]”）括起来。 相邻元素由字符“，”（逗号和空格）分隔。 
     * 通过{@link String＃valueOf（Object）}将元素转换为字符串。
     *
     * @return a string representation of this collection
     */
    public String toString() {
        Iterator<E> it = iterator();
        if (! it.hasNext())
            return "[]";

        StringBuilder sb = new StringBuilder();
        sb.append('[');
        for (;;) {
            E e = it.next();
            sb.append(e == this ? "(this Collection)" : e);
            if (! it.hasNext())
                return sb.append(']').toString();
            sb.append(',').append(' ');
        }
    }

}

```