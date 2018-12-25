```java
package java.util;

/**
 * Stack类表示对象的后进先出（LIFO）堆栈。 它使用五个操作扩展了Vector类，这些操作
 * 允许将向量视为堆栈。 提供了常见的推送和弹出操作，以及查看堆栈顶部项目的方法，测
 * 试堆栈是否为空的方法，以及搜索堆栈中的项目并发现其距离的方法 是从顶部。
 * 首次创建堆栈时，它不包含任何项目。
 *
 * {@link Deque}接口及其实现提供了一组更完整，更一致的LIFO堆栈操作，应优先使用该类。
 * 例如：
 * <pre>   {@code
 *   Deque<Integer> stack = new ArrayDeque<Integer>();}</pre>
 *
 * @author  Jonathan Payne
 * @since   JDK1.0
 */
public
class Stack<E> extends Vector<E> {
    /**
     * Creates an empty Stack.
     */
    public Stack() {
    }

    /**
     * 将项目推到此堆栈的顶部。 这与以下效果完全相同：
     * <blockquote><pre>
     * addElement(item)</pre></blockquote>
     *
     * @param   item   the item to be pushed onto this stack.
     * @return  the <code>item</code> argument.
     * @see     java.util.Vector#addElement
     */
    public E push(E item) {
        addElement(item);

        return item;
    }

    /**
     * 移除此堆栈顶部的对象，并将该对象作为此函数的值返回。
     *
     * @return  The object at the top of this stack (the last item
     *          of the <tt>Vector</tt> object).
     * @throws  EmptyStackException  if this stack is empty.
     */
    public synchronized E pop() {
        E       obj;
        int     len = size();

        obj = peek();
        removeElementAt(len - 1);

        return obj;
    }

    /**
     * 查看此堆栈顶部的对象，而不将其从堆栈中删除。
     *
     * @return  the object at the top of this stack (the last item
     *          of the <tt>Vector</tt> object).
     * @throws  EmptyStackException  if this stack is empty.
     */
    public synchronized E peek() {
        int     len = size();

        if (len == 0)
            throw new EmptyStackException();
        return elementAt(len - 1);
    }

    /**
     * 测试此堆栈是否为空。
     *
     * @return  <code>true</code> if and only if this stack contains
     *          no items; <code>false</code> otherwise.
     */
    public boolean empty() {
        return size() == 0;
    }

    /**
     * 返回对象在此堆栈上的从1开始的位置。 如果对象o作为此堆栈中的项目出现，
     * 则此方法返回距离堆栈顶部最近的堆栈顶部的距离; 堆栈中最上面的项目被认
     * 为是距离1。 equals方法用于将o与此堆栈中的项目进行比较。
     *
     * @param   o   the desired object.
     * @return  the 1-based position from the top of the stack where
     *          the object is located; the return value <code>-1</code>
     *          indicates that the object is not on the stack.
     */
    public synchronized int search(Object o) {
        int i = lastIndexOf(o);

        if (i >= 0) {
            return size() - i;
        }
        return -1;
    }

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = 1224463164541339165L;
}

```