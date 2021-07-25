## 컬렉션 프레임워크(collection framework)
### 데이터를 저장하는 자료 구조와 데이터를 처리하는 알고리즘을 구조화하여 java.util 패키지로 구현해 놓은 컨테이너 클래스.

#### 컬렉션 프레임워크 구조 및 종류
- Superinterfaces
  - Iterable
    - Collection 인터페이스에서 Iterable를 확장했는데 Iterable 인터페이스는 forEach(Consumer<? super T> action), Iterable(), spliterator() 메소드가 유일하다.
    ```
    public interface Collection<E> extends Iterable<E>
    ```
- Subinterfaces
  - BeanContext
  - BeanContextServices
  - BlockingDeque<E>
  - BlockingQueue<E> 
  - Deque<E>
  - <Strong>List<E></Strong> 
  - NavigableSet<E>
  - <Strong>Queue<E></Strong> 
  - <Strong>Set<E></Strong>
  - SortedSet<E>
  - TransferQueue<E>
- Implementing Classes  
  - AbstractCollection
  - AbstractList
  - AbstractQueue
  - AbstractSequentialList
  - AbstractSet
  - ArrayBlockingQueue
  - ArrayDeque
  - <Strong>ArrayList</Strong>
  - AttributeList
  - BeanContextServicesSupport
  - BeanContextSupport
  - ConcurrentHashMap.KeySetView
  - ConcurrentLinkedDeque
  - ConcurrentLinkedQueue
  - ConcurrentSkipListSet
  - CopyOnWriteArrayList
  - CopyOnWriteArraySet
  - DelayQueue
  - EnumSet
  - <Strong>HashSet</Strong>
  - JobStateReasons
  - LinkedBlockingDeque
  - LinkedBlockingQueue
  - <Strong>LinkedHashSet</Strong>
  - <Strong>LinkedList</Strong>
  - LinkedTransferQueue
  - PriorityBlockingQueue
  - PriorityQueue
  - RoleList
  - RoleUnresolvedList
  - <Strong>Stack</Strong>
  - SynchronousQueue
  - <Strong>TreeSet</Strong>
  - <Strong>Vector</Strong>
 
##자주쓰는컬렉션
- Interface : List, Queue, Set
- Class : ArrayList, HashSet, LinkedList, LinkedHashSet, Stack, TreeSet, Vector