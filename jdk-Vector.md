```java
 package java.util;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.StreamCorruptedException;
import java.util.function.Consumer;
import java.util.function.Predicate;
import java.util.function.UnaryOperator;

/**
 * {@code Vector}类实现了可增长的对象数组。 像数组一样，它包含可以使用整数索引
 * 访问的组件。 但是，{@code Vector}的大小可以根据需要增大或缩小，以便在创建
 * {@code Vector}后添加和删除项目。
 *
 * 每个向量都试图通过维护{@code capacity}和{@code capacityIncrement}来优化存储管理。
 * {@code capacity}始终至少与矢量大小一样大; 它通常更大，因为随着组件被添加到向量中，
 * 向量的存储以{@code capacityIncrement}的大小增加。 应用程序可以在插入大量组件之前
 * 增加向量的容量; 这减少了增量重新分配的数量。
 *
 * 此类的{@link #iterator（）iterator}和{@link #listIterator（int）listIterator}方法
 * 返回的迭代器是快速失败的</a>：如果在创建迭代器后的任何时候对向量进行结构修改 ，除
 * 了通过迭代器自己的{@link ListIterator #remove（）remove}或{@link ListIterator 
 * #add（Object）add}方法之外，迭代器将抛出{@link ConcurrentModificationException}。 
 * 因此，在并发修改的情况下，迭代器快速而干净地失败，而不是在未来的未确定时间冒任意，
 * 非确定性行为的风险。 {@link #element（）elements}方法返回的{@link Enumeration
 * Enumerations}不是快速失败的。
 *
 * 请注意，迭代器的快速失败行为无法得到保证，因为一般来说，在存在不同步的并发修改时，
 * 不可能做出任何硬性保证。 失败快速的迭代器会尽最大努力抛出
 * {@code ConcurrentModificationException}。 因此，编写依赖于此异常的程序以确保其正确
 * 性是错误的：迭代器的快速失败行为应该仅用于检测错误。
 *
 * 从Java 2平台v1.2开始，这个类被改进以实现{@link List}接口，使其成为
 * <a href =“{@ docRoot} /../ technotes / guides / collections / index的成员 .html“> 
 * Java Collections Framework </a>。 与新的集合实现不同，{@code Vector}是同步的。 如
 * 果不需要线程安全实现，建议使用{@link ArrayList}代替{@code Vector}。
 *
 * @author  Lee Boynton
 * @author  Jonathan Payne
 * @see Collection
 * @see LinkedList
 * @since   JDK1.0
 */
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    /**
     * 存储向量组件的数组缓冲区。 向量的容量是此数组缓冲区的长度，并且至少足以包含所有向量的元素。
     *
     * Vector中最后一个元素后面的任何数组元素都是null。
     *
     * @serial
     */
    protected Object[] elementData;

    /**
     * 此{@code Vector}对象中的有效组件数。 组件{@code elementData [0]}到
     * {@code elementData [elementCount-1]}是实际项目。
     *
     * @serial
     */
    protected int elementCount;

    /**
     * 当矢量大小超过其容量时，矢量容量自动递增的量。 如果容量增量小于或等于零，则每次需要
     * 增长时，矢量的容量加倍。
     *
     * @serial
     */
    protected int capacityIncrement;

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -2767605614048989439L;

    /**
     * 构造具有指定初始容量和容量增量的空向量。
     *
     * @param   initialCapacity     the initial capacity of the vector
     * @param   capacityIncrement   the amount by which the capacity is
     *                              increased when the vector overflows
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }

    /**
     * 构造一个具有指定初始容量且容量增量等于零的空向量。
     *
     * @param   initialCapacity   the initial capacity of the vector
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public Vector(int initialCapacity) {
        this(initialCapacity, 0);
    }

    /**
     * 构造一个空向量，使其内部数据数组的大小为{@code 10}，其标准容量增量为零。
     */
    public Vector() {
        this(10);
    }

    /**
     * 按照集合的迭代器返回的顺序构造一个包含指定集合元素的向量。
     *
     * @param c the collection whose elements are to be placed into this
     *       vector
     * @throws NullPointerException if the specified collection is null
     * @since   1.2
     */
    public Vector(Collection<? extends E> c) {
        elementData = c.toArray();
        elementCount = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, elementCount, Object[].class);
    }

    /**
     * 将此向量的组件复制到指定的数组中。 此向量中索引{@code k}处的项目被复制到
     * {@code anArray}的组件{@code k}中。
     *
     * @param  anArray the array into which the components get copied
     * @throws NullPointerException if the given array is null
     * @throws IndexOutOfBoundsException if the specified array is not
     *         large enough to hold all the components of this vector
     * @throws ArrayStoreException if a component of this vector is not of
     *         a runtime type that can be stored in the specified array
     * @see #toArray(Object[])
     */
    public synchronized void copyInto(Object[] anArray) {
        System.arraycopy(elementData, 0, anArray, 0, elementCount);
    }

    /**
     * 修剪此向量的容量为向量的当前大小。 如果此向量的容量大于其当前大小，则通过将字段
     * {@code elementData}中保留的内部数据数组替换为较小的容量，将容量更改为等于大小。
     * 应用程序可以使用此操作来最小化向量的存储。
     */
    public synchronized void trimToSize() {
        modCount++;
        int oldCapacity = elementData.length;
        if (elementCount < oldCapacity) {
            elementData = Arrays.copyOf(elementData, elementCount);
        }
    }

    /**
     * 如有必要，增加此向量的容量，以确保它至少可以容纳由minimum capacity参数指定的组件数。
     *
     * 如果此向量的当前容量小于{@code minCapacity}，则通过将其保留在字段{@code elementData}中
     * 的内部数据数组替换为较大的数据来增加其容量。 除非{@code capacityIncrement}的值小于或等
     * 于零，否则新数据阵列的大小将是旧大小加上{@code capacityIncrement}，在这种情况下，新容
     * 量将是旧容量的两倍; 但如果这个新大小仍小于{@code minCapacity}，则新容量将为
     * {@code minCapacity}。
     *
     * @param minCapacity the desired minimum capacity
     */
    public synchronized void ensureCapacity(int minCapacity) {
        if (minCapacity > 0) {
            modCount++;
            ensureCapacityHelper(minCapacity);
        }
    }

    /**
     * 这实现了ensureCapacity的非同步语义。 此类中的同步方法可以在内部调用此方法以确保容量，
     * 而不会产生额外同步的成本。
     *
     * @see #ensureCapacity(int)
     */
    private void ensureCapacityHelper(int minCapacity) {
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * 要分配的最大数组大小。 一些VM在阵列中保留一些标题字。 尝试分配更大的数组可能会导致
     * OutOfMemoryError：请求的数组大小超过VM限制
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    /**
     * 设置此向量的大小。 如果新大小大于当前大小，则新的{@code null}项将添加到向量的末尾。 如
     * 果新大小小于当前大小，则丢弃索引{@code newSize}和更大的所有组件。
     *
     * @param  newSize   the new size of this vector
     * @throws ArrayIndexOutOfBoundsException if the new size is negative
     */
    public synchronized void setSize(int newSize) {
        modCount++;
        if (newSize > elementCount) {
            ensureCapacityHelper(newSize);
        } else {
            for (int i = newSize ; i < elementCount ; i++) {
                elementData[i] = null;
            }
        }
        elementCount = newSize;
    }

    /**
     * 返回此向量的当前容量。
     *
     * @return  the current capacity (the length of its internal
     *          data array, kept in the field {@code elementData}
     *          of this vector)
     */
    public synchronized int capacity() {
        return elementData.length;
    }

    /**
     * 返回此向量中的组件数。
     *
     * @return  the number of components in this vector
     */
    public synchronized int size() {
        return elementCount;
    }

    /**
     * 测试此向量是否没有组件。
     *
     * @return  {@code true} if and only if this vector has
     *          no components, that is, its size is zero;
     *          {@code false} otherwise.
     */
    public synchronized boolean isEmpty() {
        return elementCount == 0;
    }

    /**
     * 返回此向量的组件的枚举。 返回的{@code Enumeration}对象将生成此向量中的所有项。
     * 生成的第一个项目是索引{@code 0}处的项目，然后是索引{@code 1}处的项目，依此类推。
     *
     * @return  an enumeration of the components of this vector
     * @see     Iterator
     */
    public Enumeration<E> elements() {
        return new Enumeration<E>() {
            int count = 0;

            public boolean hasMoreElements() {
                return count < elementCount;
            }

            public E nextElement() {
                synchronized (Vector.this) {
                    if (count < elementCount) {
                        return elementData(count++);
                    }
                }
                throw new NoSuchElementException("Vector Enumeration");
            }
        };
    }

    /**
     * 如果此向量包含指定的元素，则返回{@code true}。 更正式地，返回{@code true}当且仅当此向
     * 量包含至少一个元素{@code e}时（o == null？e == null：o.equals（e））。
     *
     * @param o element whose presence in this vector is to be tested
     * @return {@code true} if this vector contains the specified element
     */
    public boolean contains(Object o) {
        return indexOf(o, 0) >= 0;
    }

    /**
     * 返回此向量中第一次出现的指定元素的索引，如果此向量不包含该元素，则返回-1。 更正式地，返回
     * 最低索引{@code i}，使得（o == null？get（i）== null：o.equals（get（i））），如果没有这样
     * 的索引则返回-1。
     *
     * @param o element to search for
     * @return the index of the first occurrence of the specified element in
     *         this vector, or -1 if this vector does not contain the element
     */
    public int indexOf(Object o) {
        return indexOf(o, 0);
    }

    /**
     * 返回此向量中第一次出现的指定元素的索引，从{@code index}搜索向前，或者如果找不到该元素则返回-1。
     * 更正式地，返回最低索引{@code i}，使得
     *（i> =＆index（o == null？＆get（i）== null：o.equals（get（i））））</ tt> ，如果没有这样的索引，
     * 返回-1。
     *
     * @param o element to search for
     * @param index index to start searching from
     * @return the index of the first occurrence of the element in
     *         this vector at position {@code index} or later in the vector;
     *         {@code -1} if the element is not found.
     * @throws IndexOutOfBoundsException if the specified index is negative
     * @see     Object#equals(Object)
     */
    public synchronized int indexOf(Object o, int index) {
        if (o == null) {
            for (int i = index ; i < elementCount ; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = index ; i < elementCount ; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回此向量中指定元素最后一次出现的索引，如果此向量不包含该元素，则返回-1。 更正式地
     * ,返回最高索引{@code i}，使得（o == null？get（i）== null：o.equals（get（i））），如
     * 果没有这样的索引则返回-1。
     *
     * @param o element to search for
     * @return the index of the last occurrence of the specified element in
     *         this vector, or -1 if this vector does not contain the element
     */
    public synchronized int lastIndexOf(Object o) {
        return lastIndexOf(o, elementCount-1);
    }

    /**
     * 返回此向量中最后一次出现的指定元素的索引，从{@code index}向后搜索，如果找不到该元素，
     * 则返回-1。 更正式地，返回最高索引{@code i}，使得
     * （i <= index（o == null？get（i）== null：o.equals（get（i））））或-1，如果有 没有
     * 这样的指数。
     *
     * @param o element to search for
     * @param index index to start searching backwards from
     * @return the index of the last occurrence of the element at position
     *         less than or equal to {@code index} in this vector;
     *         -1 if the element is not found.
     * @throws IndexOutOfBoundsException if the specified index is greater
     *         than or equal to the current size of this vector
     */
    public synchronized int lastIndexOf(Object o, int index) {
        if (index >= elementCount)
            throw new IndexOutOfBoundsException(index + " >= "+ elementCount);

        if (o == null) {
            for (int i = index; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = index; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回指定索引处的组件。
     *
     * 此方法的功能与{@link #get（int）}方法（它是{@link List}接口的一部分）相同。
     *
     * @param      index   an index into this vector
     * @return     the component at the specified index
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= size()})
     */
    public synchronized E elementAt(int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " + elementCount);
        }

        return elementData(index);
    }

    /**
     * 返回此向量的第一个组件（索引{@code 0}处的项）。
     *
     * @return     the first component of this vector
     * @throws NoSuchElementException if this vector has no components
     */
    public synchronized E firstElement() {
        if (elementCount == 0) {
            throw new NoSuchElementException();
        }
        return elementData(0);
    }

    /**
     * 返回向量的最后一个组件。
     *
     * @return  the last component of the vector, i.e., the component at index
     *          <code>size()&nbsp;-&nbsp;1</code>.
     * @throws NoSuchElementException if this vector is empty
     */
    public synchronized E lastElement() {
        if (elementCount == 0) {
            throw new NoSuchElementException();
        }
        return elementData(elementCount - 1);
    }

    /**
     * 将此向量的指定{@code index}处的组件设置为指定对象。 该位置的前一个组件将被丢弃。
     *
     * 索引必须是大于或等于{@code 0}且小于向量当前大小的值。
     *
     * 此方法的功能与{@link #set（int，Object）set（int，E）}方法（它是{@link List}接
     * 口的一部分）相同。 请注意，{@code set}方法会反转参数的顺序，以更接近地匹配数组
     * 使用情况。 另请注意，{@code set}方法返回存储在指定位置的旧值。
     *
     * @param      obj     what the component is to be set to
     * @param      index   the specified index
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= size()})
     */
    public synchronized void setElementAt(E obj, int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        elementData[index] = obj;
    }

    /**
     * 删除指定索引处的组件。 索引中大于或等于指定{@code索引}的索引中的每个组件向下移动，
     * 使索引1小于先前的值。 该向量的大小减少{@code 1}。
     *
     * 索引必须是大于或等于{@code 0}且小于向量当前大小的值。
     *
     * 此方法的功能与{@link #remove（int）}方法（它是{@link List}接口的一部分）相同。 
     * 请注意，{@code remove}方法返回存储在指定位置的旧值。
     *
     * @param      index   the index of the object to remove
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= size()})
     */
    public synchronized void removeElementAt(int index) {
        modCount++;
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        else if (index < 0) {
            throw new ArrayIndexOutOfBoundsException(index);
        }
        int j = elementCount - index - 1;
        if (j > 0) {
            System.arraycopy(elementData, index + 1, elementData, index, j);
        }
        elementCount--;
        elementData[elementCount] = null; /* to let gc do its work */
    }

    /**
     * 将指定对象作为此向量中的组件插入指定的{@code index}。 索引中大于或等于指定{@code索引}
     * 的索引中的每个组件向上移动，使索引1大于先前的值。
     *
     * 索引必须是大于或等于{@code 0}且小于或等于向量当前大小的值。 （如果索引等于向量的当前大
     * 小，则新元素将附加到Vector。）
     *
     * 此方法的功能与{@link #add（int，Object）add（int，E）}方法（它是{@link List}接口的一部分）
     * 相同。 请注意，{@code add}方法会反转参数的顺序，以更接近地匹配数组使用情况。
     *
     * @param      obj     the component to insert
     * @param      index   where to insert the new component
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index > size()})
     */
    public synchronized void insertElementAt(E obj, int index) {
        modCount++;
        if (index > elementCount) {
            throw new ArrayIndexOutOfBoundsException(index
                                                     + " > " + elementCount);
        }
        ensureCapacityHelper(elementCount + 1);
        System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
        elementData[index] = obj;
        elementCount++;
    }

    /**
     * Adds the specified component to the end of this vector,
     * increasing its size by one. The capacity of this vector is
     * increased if its size becomes greater than its capacity.
     *
     * <p>This method is identical in functionality to the
     * {@link #add(Object) add(E)}
     * method (which is part of the {@link List} interface).
     *
     * @param   obj   the component to be added
     */
    public synchronized void addElement(E obj) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = obj;
    }

    /**
     * 将指定的组件添加到此向量的末尾，将其大小增加1。 如果该载体的大小大于其容量，则该载体的
     * 容量会增加。
     *
     * 此方法的功能与{@link #remove（Object）}方法（它是{@link List}接口的一部分）相同。
     *
     * @param   obj   the component to be removed
     * @return  {@code true} if the argument was a component of this
     *          vector; {@code false} otherwise.
     */
    public synchronized boolean removeElement(Object obj) {
        modCount++;
        int i = indexOf(obj);
        if (i >= 0) {
            removeElementAt(i);
            return true;
        }
        return false;
    }

    /**
     * 从此向量中移除所有组件并将其大小设置为零。
     *
     * 此方法的功能与{@link #clear}方法（它是{@link List}接口的一部分）相同。
     */
    public synchronized void removeAllElements() {
        modCount++;
        // Let gc do its work
        for (int i = 0; i < elementCount; i++)
            elementData[i] = null;

        elementCount = 0;
    }

    /**
     * 返回此向量的副本。 该副本将包含对内部数据数组的克隆的引用，而不是对此{@code Vector}对象
     * 的原始内部数据数组的引用。
     *
     * @return  a clone of this vector
     */
    public synchronized Object clone() {
        try {
            @SuppressWarnings("unchecked")
                Vector<E> v = (Vector<E>) super.clone();
            v.elementData = Arrays.copyOf(elementData, elementCount);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError(e);
        }
    }

    /**
     * 以正确的顺序返回包含此Vector中所有元素的数组。
     *
     * @since 1.2
     */
    public synchronized Object[] toArray() {
        return Arrays.copyOf(elementData, elementCount);
    }

    /**
     * 以正确的顺序返回包含此Vector中所有元素的数组; 返回数组的运行时类型是指定数组的运行时
     * 类型。 如果Vector适合指定的数组，则返回其中。 否则，将使用指定数组的运行时类型和此
     * Vector的大小分配新数组。
     *
     * 如果Vector适合指定的数组，并且有空余空间（即，数组的元素多于Vector），则紧跟Vector结
     * 尾的数组中的元素将设置为null。 （仅当调用者知道Vector不包含任何null元素时，这在确定
     * Vector的长度时很有用。）
     *
     * @param a the array into which the elements of the Vector are to
     *          be stored, if it is big enough; otherwise, a new array of the
     *          same runtime type is allocated for this purpose.
     * @return an array containing the elements of the Vector
     * @throws ArrayStoreException if the runtime type of a is not a supertype
     * of the runtime type of every element in this Vector
     * @throws NullPointerException if the given array is null
     * @since 1.2
     */
    @SuppressWarnings("unchecked")
    public synchronized <T> T[] toArray(T[] a) {
        if (a.length < elementCount)
            return (T[]) Arrays.copyOf(elementData, elementCount, a.getClass());

        System.arraycopy(elementData, 0, a, 0, elementCount);

        if (a.length > elementCount)
            a[elementCount] = null;

        return a;
    }

    // Positional Access Operations

    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

    /**
     * 返回此Vector中指定位置的元素。
     *
     * @param index index of the element to return
     * @return object at the specified index
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *            ({@code index < 0 || index >= size()})
     * @since 1.2
     */
    public synchronized E get(int index) {
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        return elementData(index);
    }

    /**
     * 用指定的元素替换此Vector中指定位置的元素。
     *
     * @param index index of the element to replace
     * @param element element to be stored at the specified position
     * @return the element previously at the specified position
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= size()})
     * @since 1.2
     */
    public synchronized E set(int index, E element) {
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }

    /**
     * 将指定的元素追加到此Vector的末尾。
     *
     * @param e element to be appended to this Vector
     * @return {@code true} (as specified by {@link Collection#add})
     * @since 1.2
     */
    public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }

    /**
     * 删除此向量中第一次出现的指定元素如果向量不包含该元素，则不会更改。 更正式地，
     * 删除具有最低索引i的元素，使得
     * {@code（o == null？get（i）== null：o.equals（get（i）））}（如果存在这样的元素）。
     *
     * @param o element to be removed from this Vector, if present
     * @return true if the Vector contained the specified element
     * @since 1.2
     */
    public boolean remove(Object o) {
        return removeElement(o);
    }

    /**
     * 将指定元素插入此Vector中的指定位置。 将当前位置的元素（如果有）和任何后续元素向右
     * 移动（将其添加到索引中）。
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index > size()})
     * @since 1.2
     */
    public void add(int index, E element) {
        insertElementAt(element, index);
    }

    /**
     * 删除此Vector中指定位置的元素。 将任何后续元素向左移位（从索引中减去一个元素）。 返回
     * 从Vector中删除的元素。
     *
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= size()})
     * @param index the index of the element to be removed
     * @return element that was removed
     * @since 1.2
     */
    public synchronized E remove(int index) {
        modCount++;
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);
        E oldValue = elementData(index);

        int numMoved = elementCount - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--elementCount] = null; // Let gc do its work

        return oldValue;
    }

    /**
     * 从此Vector中删除所有元素。 此调用返回后，Vector将为空（除非它抛出异常）。
     *
     * @since 1.2
     */
    public void clear() {
        removeAllElements();
    }

    // Bulk Operations

    /**
     * 如果此Vector包含指定Collection中的所有元素，则返回true。
     *
     * @param   c a collection whose elements will be tested for containment
     *          in this Vector
     * @return true if this Vector contains all of the elements in the
     *         specified collection
     * @throws NullPointerException if the specified collection is null
     */
    public synchronized boolean containsAll(Collection<?> c) {
        return super.containsAll(c);
    }

    /**
     * 将指定Collection中的所有元素追加到此Vector的末尾，按照指定Collection的Iterator返回的顺序。 
     * 如果在操作正在进行时修改了指定的Collection，则此操作的行为是不确定的。 （这意味着如果指定
     * 的Collection是此Vector，则此调用的行为是未定义的，并且此Vector非空。）
     *
     * @param c elements to be inserted into this Vector
     * @return {@code true} if this Vector changed as a result of the call
     * @throws NullPointerException if the specified collection is null
     * @since 1.2
     */
    public synchronized boolean addAll(Collection<? extends E> c) {
        modCount++;
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityHelper(elementCount + numNew);
        System.arraycopy(a, 0, elementData, elementCount, numNew);
        elementCount += numNew;
        return numNew != 0;
    }

    /**
     * 从此Vector中删除指定Collection中包含的所有元素。
     *
     * @param c a collection of elements to be removed from the Vector
     * @return true if this Vector changed as a result of the call
     * @throws ClassCastException if the types of one or more elements
     *         in this vector are incompatible with the specified
     *         collection
     * (<a href="Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if this vector contains one or more null
     *         elements and the specified collection does not support null
     *         elements
     * (<a href="Collection.html#optional-restrictions">optional</a>),
     *         or if the specified collection is null
     * @since 1.2
     */
    public synchronized boolean removeAll(Collection<?> c) {
        return super.removeAll(c);
    }

    /**
     * 仅保留此Vector中包含在指定Collection中的元素。 换句话说，从此Vector中删除未包
     * 含在指定Collection中的所有元素。
     *
     * @param c a collection of elements to be retained in this Vector
     *          (all other elements are removed)
     * @return true if this Vector changed as a result of the call
     * @throws ClassCastException if the types of one or more elements
     *         in this vector are incompatible with the specified
     *         collection
     * (<a href="Collection.html#optional-restrictions">optional</a>)
     * @throws NullPointerException if this vector contains one or more null
     *         elements and the specified collection does not support null
     *         elements
     *         (<a href="Collection.html#optional-restrictions">optional</a>),
     *         or if the specified collection is null
     * @since 1.2
     */
    public synchronized boolean retainAll(Collection<?> c) {
        return super.retainAll(c);
    }

    /**
     * 将指定Collection中的所有元素插入到此Vector中的指定位置。 将当前位置的元素（如果有）和任
     * 何后续元素向右移动（增加其索引）。 新元素将按照指定Collection的迭代器返回的顺序出现在
     * Vector中。
     *
     * @param index index at which to insert the first element from the
     *              specified collection
     * @param c elements to be inserted into this Vector
     * @return {@code true} if this Vector changed as a result of the call
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index > size()})
     * @throws NullPointerException if the specified collection is null
     * @since 1.2
     */
    public synchronized boolean addAll(int index, Collection<? extends E> c) {
        modCount++;
        if (index < 0 || index > elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityHelper(elementCount + numNew);

        int numMoved = elementCount - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        elementCount += numNew;
        return numNew != 0;
    }

    /**
     * 将指定的Object与此Vector进行比较以获得相等性。 当且仅当指定的Object也是List时，
     * 返回true，两个列表具有相同的大小，并且两个列表中的所有对应元素对都相等。 
     * （如果{@code（e1 == null？e2 == null：e1.equals（e2））}，则两个元素{@code e1}
     * 和{@code e2}相等。）换句话说，两个元素定义为 如果它们包含相同顺序的相同元素，则
     * 相等。
     *
     * @param o the Object to be compared for equality with this Vector
     * @return true if the specified Object is equal to this Vector
     */
    public synchronized boolean equals(Object o) {
        return super.equals(o);
    }

    /**
     * Returns the hash code value for this Vector.
     */
    public synchronized int hashCode() {
        return super.hashCode();
    }

    /**
     * Returns a string representation of this Vector, containing
     * the String representation of each element.
     */
    public synchronized String toString() {
        return super.toString();
    }

    /**
     * 返回此List的部分在fromIndex（包含）和toIndex（独占）之间的视图。 
     *（如果fromIndex和toIndex相等，则返回的List为空。）返回的List由此List支持，因此返
     * 回List中的更改将反映在此List中，反之亦然。 返回的List支持此List支持的所有可选
     * List操作。
     *
     * 此方法消除了对显式范围操作的需要（对于数组通常存在的排序）。 任何期望List的操作都
     * 可以通过操作subList视图而不是整个List来用作范围操作。 例如，以下习语从List中删除了
     * 一系列元素：
     * <pre>
     *      list.subList(from, to).clear();
     * </pre>
     * 可以为indexOf和lastIndexOf构造类似的习语，并且可以将Collections类中的所有算法应
     * 用于subList。
     *
     * 如果支持列表（即此List）在结构上以除了返回List之外的任何方式进行修改，则此方法返回
     * 的List的语义将变为未定义。 （结构修改是那些改变列表大小的修改，或以其他方式扰乱它，
     * 使得正在进行的迭代可能产生不正确的结果。）
     *
     * @param fromIndex low endpoint (inclusive) of the subList
     * @param toIndex high endpoint (exclusive) of the subList
     * @return a view of the specified range within this List
     * @throws IndexOutOfBoundsException if an endpoint index value is out of range
     *         {@code (fromIndex < 0 || toIndex > size)}
     * @throws IllegalArgumentException if the endpoint indices are out of order
     *         {@code (fromIndex > toIndex)}
     */
    public synchronized List<E> subList(int fromIndex, int toIndex) {
        return Collections.synchronizedList(super.subList(fromIndex, toIndex),
                                            this);
    }

    /**
     * 从此列表中删除索引在{@code fromIndex}之间的所有元素，包括{@code toIndex}，独占。 将任何
     * 后续元素向左移动（降低其索引）。 此调用通过{@code（toIndex  -  fromIndex）}元素缩短列表。 
     *（如果{@code toIndex == fromIndex}，此操作无效。）
     */
    protected synchronized void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = elementCount - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        // Let gc do its work
        int newElementCount = elementCount - (toIndex-fromIndex);
        while (elementCount != newElementCount)
            elementData[--elementCount] = null;
    }

    /**
     * 从流中加载{@code Vector}实例（即反序列化它）。 此方法执行检查以确保字段的一致性。
     *
     * @param in the stream
     * @throws java.io.IOException if an I/O error occurs
     * @throws ClassNotFoundException if the stream contains data
     *         of a non-existing class
     */
    private void readObject(ObjectInputStream in)
            throws IOException, ClassNotFoundException {
        ObjectInputStream.GetField gfields = in.readFields();
        int count = gfields.get("elementCount", 0);
        Object[] data = (Object[])gfields.get("elementData", null);
        if (count < 0 || data == null || count > data.length) {
            throw new StreamCorruptedException("Inconsistent vector internals");
        }
        elementCount = count;
        elementData = data.clone();
    }

    /**
     * 将{@code Vector}实例的状态保存到流中（即将其序列化）。 此方法执行同步以确保序列
     * 化数据的一致性。
     */
    private void writeObject(java.io.ObjectOutputStream s)
            throws java.io.IOException {
        final java.io.ObjectOutputStream.PutField fields = s.putFields();
        final Object[] data;
        synchronized (this) {
            fields.put("capacityIncrement", capacityIncrement);
            fields.put("elementCount", elementCount);
            data = elementData.clone();
        }
        fields.put("elementData", data);
        s.writeFields();
    }

    /**
     * 从列表中的指定位置开始，返回列表中元素的列表迭代器（按正确顺序）。 指定的索引表示初始
     * 调用{@link ListIterator＃next next}将返回的第一个元素。 对
     * {@link ListIterator＃previous previous}的初始调用将返回指定索引减去1的元素。
     *
     * <p>The returned list iterator is <a href="#fail-fast"><i>fail-fast</i></a>.
     *
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public synchronized ListIterator<E> listIterator(int index) {
        if (index < 0 || index > elementCount)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }

    /**
     * 返回此列表中元素的列表迭代器（按适当顺序）。
     *
     * <p>The returned list iterator is <a href="#fail-fast"><i>fail-fast</i></a>.
     *
     * @see #listIterator(int)
     */
    public synchronized ListIterator<E> listIterator() {
        return new ListItr(0);
    }

    /**
     * 以适当的顺序返回此列表中元素的迭代器。
     *
     * 返回的迭代器是<a href="#fail-fast">失败快速</a>。
     *
     * @return an iterator over the elements in this list in proper sequence
     */
    public synchronized Iterator<E> iterator() {
        return new Itr();
    }

    /**
     * AbstractList.Itr的优化版本
     */
    private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        public boolean hasNext() {
            // Racy but within spec, since modifications are checked
            // within or after synchronization in next/previous
            return cursor != elementCount;
        }

        public E next() {
            synchronized (Vector.this) {
                checkForComodification();
                int i = cursor;
                if (i >= elementCount)
                    throw new NoSuchElementException();
                cursor = i + 1;
                return elementData(lastRet = i);
            }
        }

        public void remove() {
            if (lastRet == -1)
                throw new IllegalStateException();
            synchronized (Vector.this) {
                checkForComodification();
                Vector.this.remove(lastRet);
                expectedModCount = modCount;
            }
            cursor = lastRet;
            lastRet = -1;
        }

        @Override
        public void forEachRemaining(Consumer<? super E> action) {
            Objects.requireNonNull(action);
            synchronized (Vector.this) {
                final int size = elementCount;
                int i = cursor;
                if (i >= size) {
                    return;
                }
        @SuppressWarnings("unchecked")
                final E[] elementData = (E[]) Vector.this.elementData;
                if (i >= elementData.length) {
                    throw new ConcurrentModificationException();
                }
                while (i != size && modCount == expectedModCount) {
                    action.accept(elementData[i++]);
                }
                // update once at end of iteration to reduce heap write traffic
                cursor = i;
                lastRet = i - 1;
                checkForComodification();
            }
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

    /**
     * AbstractList.ListItr的优化版本
     */
    final class ListItr extends Itr implements ListIterator<E> {
        ListItr(int index) {
            super();
            cursor = index;
        }

        public boolean hasPrevious() {
            return cursor != 0;
        }

        public int nextIndex() {
            return cursor;
        }

        public int previousIndex() {
            return cursor - 1;
        }

        public E previous() {
            synchronized (Vector.this) {
                checkForComodification();
                int i = cursor - 1;
                if (i < 0)
                    throw new NoSuchElementException();
                cursor = i;
                return elementData(lastRet = i);
            }
        }

        public void set(E e) {
            if (lastRet == -1)
                throw new IllegalStateException();
            synchronized (Vector.this) {
                checkForComodification();
                Vector.this.set(lastRet, e);
            }
        }

        public void add(E e) {
            int i = cursor;
            synchronized (Vector.this) {
                checkForComodification();
                Vector.this.add(i, e);
                expectedModCount = modCount;
            }
            cursor = i + 1;
            lastRet = -1;
        }
    }

    @Override
    public synchronized void forEach(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        final int expectedModCount = modCount;
        @SuppressWarnings("unchecked")
        final E[] elementData = (E[]) this.elementData;
        final int elementCount = this.elementCount;
        for (int i=0; modCount == expectedModCount && i < elementCount; i++) {
            action.accept(elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public synchronized boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        // figure out which elements are to be removed
        // any exception thrown from the filter predicate at this stage
        // will leave the collection unmodified
        int removeCount = 0;
        final int size = elementCount;
        final BitSet removeSet = new BitSet(size);
        final int expectedModCount = modCount;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            @SuppressWarnings("unchecked")
            final E element = (E) elementData[i];
            if (filter.test(element)) {
                removeSet.set(i);
                removeCount++;
            }
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }

        // shift surviving elements left over the spaces left by removed elements
        final boolean anyToRemove = removeCount > 0;
        if (anyToRemove) {
            final int newSize = size - removeCount;
            for (int i=0, j=0; (i < size) && (j < newSize); i++, j++) {
                i = removeSet.nextClearBit(i);
                elementData[j] = elementData[i];
            }
            for (int k=newSize; k < size; k++) {
                elementData[k] = null;  // Let gc do its work
            }
            elementCount = newSize;
            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
            modCount++;
        }

        return anyToRemove;
    }

    @Override
    @SuppressWarnings("unchecked")
    public synchronized void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        final int expectedModCount = modCount;
        final int size = elementCount;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            elementData[i] = operator.apply((E) elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }

    @SuppressWarnings("unchecked")
    @Override
    public synchronized void sort(Comparator<? super E> c) {
        final int expectedModCount = modCount;
        Arrays.sort((E[]) elementData, 0, elementCount, c);
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }

    /**
     * 对此列表中的元素创建<a href="Spliterator.html#binding">后期绑定</a>和失败快速{@link Spliterator}。
     *
     * {@code Spliterator}报告{@link Spliterator＃SIZED}，{@ link Spliterator＃SUBSIZED}和
     * {@link Spliterator＃ORDERED}。 覆盖实现应记录其他特征值的报告。
     *
     * @return a {@code Spliterator} over the elements in this list
     * @since 1.8
     */
    @Override
    public Spliterator<E> spliterator() {
        return new VectorSpliterator<>(this, null, 0, -1, 0);
    }

    /** Similar to ArrayList Spliterator */
    static final class VectorSpliterator<E> implements Spliterator<E> {
        private final Vector<E> list;
        private Object[] array;
        private int index; // current index, modified on advance/split
        private int fence; // -1 until used; then one past last index
        private int expectedModCount; // initialized when fence set

        /** Create new spliterator covering the given  range */
        VectorSpliterator(Vector<E> list, Object[] array, int origin, int fence,
                          int expectedModCount) {
            this.list = list;
            this.array = array;
            this.index = origin;
            this.fence = fence;
            this.expectedModCount = expectedModCount;
        }

        private int getFence() { // initialize on first use
            int hi;
            if ((hi = fence) < 0) {
                synchronized(list) {
                    array = list.elementData;
                    expectedModCount = list.modCount;
                    hi = fence = list.elementCount;
                }
            }
            return hi;
        }

        public Spliterator<E> trySplit() {
            int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
            return (lo >= mid) ? null :
                new VectorSpliterator<E>(list, array, lo, index = mid,
                                         expectedModCount);
        }

        @SuppressWarnings("unchecked")
        public boolean tryAdvance(Consumer<? super E> action) {
            int i;
            if (action == null)
                throw new NullPointerException();
            if (getFence() > (i = index)) {
                index = i + 1;
                action.accept((E)array[i]);
                if (list.modCount != expectedModCount)
                    throw new ConcurrentModificationException();
                return true;
            }
            return false;
        }

        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> action) {
            int i, hi; // hoist accesses and checks from loop
            Vector<E> lst; Object[] a;
            if (action == null)
                throw new NullPointerException();
            if ((lst = list) != null) {
                if ((hi = fence) < 0) {
                    synchronized(lst) {
                        expectedModCount = lst.modCount;
                        a = array = lst.elementData;
                        hi = fence = lst.elementCount;
                    }
                }
                else
                    a = array;
                if (a != null && (i = index) >= 0 && (index = hi) <= a.length) {
                    while (i < hi)
                        action.accept((E) a[i++]);
                    if (lst.modCount == expectedModCount)
                        return;
                }
            }
            throw new ConcurrentModificationException();
        }

        public long estimateSize() {
            return (long) (getFence() - index);
        }

        public int characteristics() {
            return Spliterator.ORDERED | Spliterator.SIZED | Spliterator.SUBSIZED;
        }
    }
}

```