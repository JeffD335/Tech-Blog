For this LinkedList(LinkedListDeque) I implemented, it's implemented by inner class ``ListNode`` . The whole LinkedListDeque class have one variable ``size`` and an invariant ``ListNode sentinel`` and it is circular which means it acts like two sentinel in front and back, but in fact just one of it. Since the ``sentinel.next`` always points to the first node and the sentinel.prev always points to the last node, so the ``addFirst``, ``addLast``, ``removeFirst``, ``removeLast `` operations can be done in O(1) time complexity.

For ArrayDeque, it was implemented by using resizable cicular array. It maintains two invariants ``nextFirst`` and ``nextLast``: ``nextFirst`` points to empty space in front of first actual element, while the ``nextLast`` points to the empty space behind the last actual element. Using modulo operation mapping logical index to real array index, realizing array reusing. So that ``addFirst``, ``addLast``, ``removeFirst``, ``removeLast`` operations do not need to move all the elements around and realize amortized O(1).

In addition, It also achieves automatic capacity expansion and contraction: expansion when it about full, contraction when the logical size is way smaller than real length of the array. So that it making sure that add/ remove operations amortized O(1), and avoiding occupation excessive memory after deleting a large number of elements.

Besides, I implemented equals and iterator. The equals method compares the logical sequences of two deques, so deques with different underlying implementations should be equal as long as their element orders are the same. The iterator method enables deques to support for-each traversal.

**Simple Deque Interface** 

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

**LinkedListDeque**

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

**ArrayDeque**

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

