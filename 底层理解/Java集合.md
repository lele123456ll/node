# 集合

- List: 存储元素是有序的、可重复的
- Set : 存储元素是无序的、不可重复的
- Map : 使用键值对key-value存储， key是无序的不可重复的， value是无序的可重复的

## List

- 实现类

    - ArrayList

        ```java
        1.线程不安全, 没有同步函数
        2. 底层实现 --> Object数组
        return new Object[]
        3. 插入删除
            受到元素位置影响
        4. 内存空间占用
            ArrayList中空间浪费在list列表回留有一定的容量空间
        ```

    - LinkedList

        ```
        1. 线程不安全, 没有同步函数
        2. 底层实现 --> 双向链表
        3. 插入删除
            插入: O(1), 删除: O(n)
        4. 内存空间占用
            每一个元素需要消耗更多的空间(前驱,后继)
        ```

    - Vector

        ```
        1. 线程安全, 有synchronize函数
        2. 底层实现 --> Object数组
        3. 插入删除
        	受到元素位置影响
        ```

- ArrayList扩容

    ```java
    1. 添加元素--> add()
    public boolean add(E e) {
        ensureCapacityInternal(size + 1); 
        elementData[size++] = e;
        return true;
    }
    2. 确保内部容量
    private void ensureCapacityInternal(int minCapacity) {
        // 如果当前Object数组为空, 设置容量为DEFAULT_CAPACITY=10
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        // Object不为空
        ensureExplicitCapacity(minCapacity);
    }
    3. 确保显性容量
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++; // 记录修改次数
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity); // 扩容
    }
    4. 扩容
    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        // 扩容后 = 原来容量 + 原来容量/2
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // 如果扩容后还是小于minCapacity, 直接赋值为minCapacity
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // 如果扩容后容量大于数组最大容量, MAX_ARRAY_SIZE = 0x7fffffff - 8
        // hugeCapacity设置newCapacity为Integer.MAX_VALUE
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }   
    ```

    - 为什么数组MAX_ARRAY_SIZE = 0x7fffffff - 8 ?
        - 在数组的对象头里有一个length字段，记录数组长度，只需要去读length字段就可以了。这个8就是就是存了数组length字段。

## Set

### 特性与实现类

- **存储无序的、不可重复的数据 --\>高中讲的"集合"**
- 实现类
    - **HashSet**
    - LinkedHashSet
    - TreeSet
- **HashSet底层:数组+链表的结构。**

### HashSet底层

1. 无序性:不等于随机性。存储的数据在底层数组中并非按照数组索引的顺序添加，而是根据数据的哈希值
2. 不可重复性:保证添加的元素按照equals()判断时，不能返回true. 即:**相同的元素只能添加一个**。

### 添加元素的过程(HashSet ):

- 我们向HashSet中添加元素a,首先调用元素a所在类的hashCode()方法，**计算元素a的哈希值**,此哈希值接着通过某种算法**计算出在HashSet底层数组中的存放位置(即为:索引位置)**
- 判断数组此位置上是否已经有元素:
    - 如果此位置上没有其他元素，则元素a添加成功 --> **情况1**
    - 如果此位置上**有其他元素b**(或以链表形式存在的多个元素)，则**比较元素a与元素b的hash值**:
        - 如果**hash值不相同**，则元素a**添加成功**。--\> 情况2
        - 如果**hash值相同**，进而需要调用元素a所在类的**equals()方法**:
            - **equals()返回true**,元素a**添加失败**
            - **equals()返回false**,则元素a**添加成功**。--\>  **情况2**
- 对于添加成功的情况2和情况3而言:元素a与已经存在指定索引位置上数据以链表的方式存储。

> jdk 7 :元素a放到数组中，指向原来的元素。
> jdk 8 :原来的元素在数组中，指向元素a

## Map

### 实现

- **HashMap**
    - 作为Map的主要实现类;**线程不安全的，效率高;**存储nuLL的key和value
- **LinkedHashMap**
    - 保证在遍历map元素时，可以按照添加的顺序实现遍历。
    - 在原有的HashMap底层结构基础上，添加了一对指针，指向前一个和后一个元素.
    - 对于频繁的遍历操作，此类执行效率高于HashMap。

- **TreeMap**
    - 保证按照添加的key-value对进行排序，实现排序遍历。此时考虑key的**自然排序或定制排序**,**底层使用红黑树**

- Hashtable:作为古老的实现类;线程安全的，效率低;不能存储null的key和value

- Properties: 常用来**处理配置文件**。key和value 都是String类型

### HashMap底层

> JDK8之前
>
> HashMap map = new HashMap( ):
> 在实例化以后，底层创建了长度是16的一维数组Entry[] table。
> ...可能已经执行过多次put...
> map.put(key1, value1)

-  **首先**，调用key1所在类的hashCode() **计算key1哈希值**，此哈希值经过某种算法计算以后，**得到在Entry 数组中的存放位置。**

    - 如果**此位置上的数据为空**，此时的key1-value1 **添加成功**。---- 情况1

    - 如果**此位置上的数据不为空**，( 意味着此位置上存在一个或多个数据(以链表形式存在)),**比较key1和已经存在的一个或多个数据的哈希值**:
        - 如果**key1的哈希值与已经存在的数据的哈希值都不相同**，此时key1-value1 **添加成功**---- 情况2
        - 如果**key1的哈希值和已经存在的某一个数据(key2-value2) 的哈希值相同**
            - **调用key1 所在类的equals(key2)**
                - 如果**equals()返回false**:此时key1-value1**添加成功**。---- 情况3
            - 如果**equals()返回true**:使用value1替换vaLue2。

> JDK8之后

1. new HashMap():底层**没有创建一个长度为16的数组**
2. jdk 8底层的数组是: **Node[], 而非Entry[]**
3. **首次调用put()方法**时，底层**创建长度为16的数组**
4. jdk7底层结构只有:数组+链表。jdk8 中底层结构:**数组+链表+红黑树。**

- **重要的常量**

    ```
    DEFAULT_INITIAL_CAPACITY : HashMap 的默认容量，16
    DEFAULT_LOAD_FACTOR: HashMap的默认加载因子: 0.75r
    threshold:扩容的临界值，= 容量*填充因子: 16 * 0.75 => 12
    TREEIFY_THRESHOLD: Bucket 中链表长度大于该默认值，转化为红黑树:8
    MIN_TREEIFY_CAPACITY:桶中的Node被树化时最小的hash表容量:64
    ```

    

