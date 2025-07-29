- Iterator 和 ListIterator 都是接口；
- ListIterator **继承自** Iterator 接口；
- ArrayList 通过内部类 Itr 和 ListItr 来实现这两个接口；
- ArrayList 提供了两个方法来获取迭代器：
  - iterator()：返回一个 Itr（实现 Iterator）
  - listIterator() / listIterator(int index)：返回一个 ListItr（实现 ListIterator）



**你需要注意的一点：**

- 并不是 ListIterator 实现了 Iterator，而是 ListIterator **继承了** Iterator 接口。
- ArrayList 本身并不实现这两个接口，它是通过内部类 Itr 和 ListItr 去分别实现它们。



### **Iterator 接口：**

```java
public interface Iterator<E> {
    boolean hasNext();
    E next();
    default void remove(); // JDK 8起是default方法
}
```



### **ListIterator 接口：**

```java
public interface ListIterator<E> extends Iterator<E> {
    boolean hasPrevious();
    E previous();
    int nextIndex();
    int previousIndex();
    void set(E e);
    void add(E e);
}
```



### **ArrayList 中迭代器的内部实现结构：**

```java
public class ArrayList<E> {
  	private int size;
  
    private Object[] elementData;

    public Iterator<E> iterator() {
        return new Itr(); // 返回内部类 Itr 的实例
    }
  
    private class Itr implements Iterator<E> {
        int cursor;       // 下一个要返回的元素下标
        int lastRet = -1; // 上一次返回的元素下标
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size;
        }

        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet); // 底层调用 ArrayList 的 remove
        				cursor = lastRet;               // 删除后，游标回退一步
        				lastRet = -1;                   // 重置上次访问位置
        				expectedModCount = modCount;   // 同步修改次数
            } catch (IndexOutOfBoundsException ex) {
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
            super();
            cursor = index;
        }

        public boolean hasPrevious() {
            return cursor != 0;
        }

        public E previous() {
            checkForComodification();
            int i = cursor - 1;
            if (i < 0)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i;
            return (E) elementData[lastRet = i];
        }

        public int nextIndex() {
            return cursor;
        }

        public int previousIndex() {
            return cursor - 1;
        }

        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.set(lastRet, e);
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                ArrayList.this.add(i, e);
                cursor = i + 1;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }

    public Iterator<E> iterator() {
        return new Itr();
    }

    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }

    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: " + index);
        return new ListItr(index);
    }
}
```



**总结：**

| **接口**     | **关键方法**                               | **是否支持双向遍历** | **是否支持插入/设置** |
| ------------ | ------------------------------------------ | -------------------- | --------------------- |
| Iterator     | hasNext(), next(), remove()                | ❌                    | ❌                     |
| ListIterator | hasPrevious(), previous(), add(), set() 等 | ✅                    | ✅                     |

- ArrayList.iterator() 返回一个只支持向前遍历、删除的迭代器（Itr）；
- ArrayList.listIterator() 返回一个支持前后遍历、插入、设置的迭代器（ListItr）；
- 都是**内部类实现**的，依赖 modCount 实现 fail-fast；
- 不管是哪种迭代器，操作过程中结构被外部修改都会抛出 ConcurrentModificationException。



### 为什么Iterator可以安全的remove

remove有一步

```java
lastRet = -1;                   // 重置上次访问位置
```

因为Iterator.remove()只能删除当前访问到的这个elementdata，所以游标回退就能实现remove之后安全的继续next。



### 增强for循环

```java
for (Integer a : nums) {
    System.out.println(a);
}
```

等效于下面这种写法

```java
for (Iterator<Integer> it = nums.iterator(); it.hasNext(); ) {
    Integer a = it.next();
    System.out.println(a);
}
```



| **语法**               | **实际等效**                                                 |
| ---------------------- | ------------------------------------------------------------ |
| for (T e : collection) | Iterator<T> it = collection.iterator(); while (it.hasNext()) { T e = it.next(); ... } |
| 本质                   | 编译器自动使用了迭代器                                       |
| 优点                   | 简洁、可读性好                                               |
| 缺点                   | 无法手动控制索引、无法使用 ListIterator 特有的方法           |