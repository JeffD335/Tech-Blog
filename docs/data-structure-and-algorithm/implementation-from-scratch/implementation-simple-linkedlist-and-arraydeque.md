## Implementation Simple LinkedList and ArrayDeque



在 proj1 中，我实现了两种 Deque。

LinkedListDeque 是用双向链表实现的。每个节点由内部类 ListNode 表示，包含 item、prev 和 next。我使用 circular sentinel，让 sentinel 的 prev 和 next 在空链表时都指向自己。这样可以避免 null 特判，也不需要额外维护 first 和 last 指针。由于 sentinel.next 永远是第一个节点，sentinel.prev 永远是最后一个节点，所以 addFirst、addLast、removeFirst、removeLast 都可以在 O(1) 时间完成。

ArrayDeque 是用 resizable circular array 实现的。它维护 nextFirst 和 nextLast 两个不变量：nextFirst 指向第一个元素前面的空位，nextLast 指向最后一个元素后面的空位。通过取模运算把逻辑 index 映射到底层数组 index，从而实现数组的循环使用。这样 addFirst、addLast、removeFirst、removeLast 不需要整体移动元素，能够达到 amortized O(1)。

此外，我实现了 equals 和 iterator。equals 比较的是两个 deque 的逻辑序列，因此不同底层实现的 deque 只要元素顺序相同也应该相等。iterator 让 deque 支持 for-each 遍历。

ArrayDeque 还实现了自动扩容和缩容：满时扩容，使用率过低时缩容。这样既能保证操作的 amortized O(1)，又能避免删除大量元素后继续占用过多内存。



```java
public interface Deque<T> {
    void addFirst(T item);

    void addLast(T item);

    default boolean isEmpty(){
        return size() == 0;
    }

    int size();

    void printDeque();

    T removeFirst();

    T removeLast();

    T get(int index);
}
```



```java
import java.util.Iterator;
import java.util.Objects;

public class LinkedListDeque<T> implements Deque<T>, Iterable<T> {
    private ListNode sentinel;
    private int size;

    private class ListNode {
        public T item;
        public ListNode prev;
        public ListNode next;
//        Constructor
        public ListNode(T x, ListNode prev, ListNode next){
            this.item = x;
            this.prev = prev;
            this.next = next;
        }
    }

    /**
     * Empty constructor:
     * create a empty Deque
     */
    public LinkedListDeque(){
        sentinel = new ListNode(null, null, null);
        sentinel.next = sentinel;
        sentinel.prev = sentinel;
        size = 0;
    }

    /**
     *  get size of the list in constant time
     * @return size
     */
  	@Override
    public int size(){
        return size;
    }
		@Override
    public void addFirst(T x) {
        ListNode listNode = new ListNode((T) x, sentinel, sentinel.next);
        sentinel.next.prev = listNode;
        sentinel.next = listNode;
        size++;
    }
		@Override
    public void addLast(T x) {
        ListNode listNode = new ListNode((T) x, sentinel.prev, sentinel);
        sentinel.prev.next = listNode;
        sentinel.prev = listNode;
        size++;
    }
		@Override
    public T removeFirst() {
        if(size >= 1){
            ListNode firstNode = sentinel.next;
            ListNode nextNode = sentinel.next.next;
            sentinel.next = nextNode;
            nextNode.prev = sentinel;
            size--;
            return firstNode.item;
        }
        return null;
    }
		@Override
    public T removeLast() {
        if(size >= 1){
            ListNode lastNode = sentinel.prev;
            lastNode.prev.next = sentinel;
            sentinel.prev = lastNode.prev;
            size--;
            return lastNode.item;
        }
        return null;
    }

    /**
     * Iterative get the node's item
     * @return item
     */
  	@Override
    public T get(int index) {
        if (size > 0 && sentinel.next != null && size > index && index >= 0){
            ListNode result = sentinel.next;
            while(index > 0){
                result = result.next;
                index--;
            }
            return result.item;
        }
        return null;
    }

    /**
     * Recursive get the node's item
     * @param index
     * @return item
     */
    public T getRecursive(int index) {
        if(size >= 1 && size > index && index >= 0){
            return getRecursiveHelper(sentinel.next, index);
        }
        return null;
    }
    private T getRecursiveHelper(ListNode node, int index){
        if(index == 0) {
            return node.item;
        }
        else{
            return getRecursiveHelper(node.next, index - 1);
        }
    }
		@Override
    public void printDeque() {
        if(size > 0) {
            int times = size;
            ListNode printNode = sentinel.next;
            while (times > 0) {
                System.out.print(printNode.item + " ");
                printNode = printNode.next;
                times--;
            }
            System.out.println();
        }
    }
    @Override
    public Iterator<T> iterator() {
        return new LinkedListDequeIterator();
    }

    private class LinkedListDequeIterator implements Iterator<T>{
        private int index;
        public LinkedListDequeIterator(){
            index = 0;
        }
        @Override
        public boolean hasNext() {
            return index < size;
        }
        @Override
        public T next() {
            T returnItem = get(index);
            index++;
            return returnItem;
        }
    }

    @Override
    public boolean equals(Object o){
        if (this == o) {
            return true;
        }
        if (o instanceof Deque){
            Deque<T> otherDeque = (Deque<T>) o;
            if (this.size() != otherDeque.size()){
                return false;
            }
            for (int i = 0; i < this.size(); i++){
                if(!Objects.equals(this.get(i), otherDeque.get(i))){
                    return false;
                }
            }
            return true;
        }
        return false;
    }
}
```



```java
package deque;


import java.util.Iterator;
import java.util.Objects;

public class ArrayDeque<T> implements Deque<T>, Iterable<T> {
    private T[] item;
    private int size;
    private int nextFirst;
    private int nextLast;


    public ArrayDeque() {
        item = (T[]) new Object[8];
        size = 0;
        nextFirst = 0;
        nextLast = 1;
    }
    @Override
    public void addFirst(T x) {
        ifFullResizing();
        item[nextFirst] = (T) x;
        nextFirst = leftShift( nextFirst);
        size++;
        ifFullResizing();
    }
  	@Override
    public void addLast(T x) {
        ifFullResizing();
        item[nextLast] = (T) x;
        nextLast = rightShift(nextLast);
        size++;
        ifFullResizing();
    }
  	@Override
    public T removeFirst() {
        if(size > 0) {
            T removedElement = item[rightShift((nextFirst))];
            nextFirst = rightShift(nextFirst);
            size--;
            sizeDownIfNeeded();
            return removedElement;
        }
        return null;
    }
		@Override
    public T removeLast() {
        if(size > 0) {
            T removedElement = item[leftShift(nextLast)];
            nextLast = leftShift(nextLast);
            size--;
            sizeDownIfNeeded();
            return removedElement;
        }
        return null;
    }
		@Override
    public int size() {
        return size;
    }
		@Override
    public void printDeque() {
        for (int i = 0; i < size; i++) {
            System.out.print(get(i));
        }
        System.out.println();
    }
    /**
     * Get the element in logical order
     * @param index
     * @return
     */
  	@Override
    public T get(int index) {
        if(index >= size || index < 0) {
            return null;
        }
        int realIndex = (nextFirst + 1 + index) % item.length;
        return item[realIndex];
    }
    /**
     * These two function are using to shift nextFirst and nextLast after add or remove operations.
     *
     */
    private int leftShift(int index) {
        return (index - 1 + item.length) % item.length;
    }
    private int rightShift(int index) {
        return (index + 1) % item.length;
    }
		
    private boolean isFull() {
        return size >= item.length;
    }

    private void ifFullResizing() {
        if (isFull()) {
            resizing();
        }
    }
    private void sizeDownIfNeeded() {
        if (item.length * 0.25 > size) {
            resizingDown();
        }
    }

    private void resizing() {
        T[] newArr = (T[]) new Object[size * 2];
        for (int i = 0; i < size; i++) {
            newArr[i] = get(i);
        }
        item = newArr;
        nextFirst = item.length - 1;
        nextLast = size;
    }

    private void resizingDown() {
        T[] newArr = (T[]) new Object[item.length / 2];
        for (int i = 0; i < size; i++) {
            newArr[i] = get(i);
        }
        item = newArr;
        nextFirst = item.length - 1;
        nextLast = size;
    }

    @Override
    public Iterator<T> iterator() {
        return new ArrayDequeIterator();
    }

    private class ArrayDequeIterator implements Iterator<T>{
        private int index;
        public ArrayDequeIterator(){
            index = 0;
        }
        @Override
        public boolean hasNext() {
            return index < size;
        }
        @Override
        public T next() {
            T returnItem = get(index);
            index++;
            return returnItem;
        }
    }
    @Override
    public boolean equals(Object o){
        if (this == o) {
            return true;
        }
        if (o instanceof Deque){
           Deque<T> otherDeque = (Deque<T>) o;
           if (this.size() != otherDeque.size()){
               return false;
           }
           for (int i = 0; i < this.size(); i++){
               if(!Objects.equals(this.get(i), otherDeque.get(i))){
                   return false;
               }
           }
           return true;
        }
        return false;
    }
}
```

