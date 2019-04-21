# ArrayList分析


----------
## ArrayList概述 ##

 - 动态增长和缩减的索引序列，它是基于数组实现的List类
 - 封装了一个动态的object[]数组(即可以存放所有类型数据)，类对象有一个capacity属性，表示它们所封装的Object[]数组的长度。
 - 线程不安全
## ArrayList源码 ##
 - 层次关系：
    ![](/images/Arraylist_01.gif)
    ![](/images/Arraylist_02.gif)
    1）为什么要先继承AbstractList，而让AbstractList先实现List<E>？而不是让ArrayList直接实现List<E>？
    ：接口中全都是抽象的方法，而抽象类中可以有抽象方法，还可以有具体的实现方法，正是利用了这一点，让AbstractList是实现接口中一些通用的方法，而具体的类，如ArrayList就继承这个AbstractList类，拿到一些通用的方法，然后自己在实现一些自己特有的方法，这样一来，让代码更简洁，就继承结构最底层的类中通用的方法都抽取出来，先一起实现了，**减少重复代码**。所以一般看到一个类上面还有一个抽象类，应该就是这个作用。
    2）ArrayList实现了哪些接口？
    ：**RandomAccess接口**：这个是一个标记性接口，通过查看api文档，它的作用就是用来快速随机存取，有关效率的问题，在实现了该接口的话，那么**使用普通的for循环来遍历，性能更高**，例如arrayList。而没有实现该接口的话，使用Iterator来迭代，这样性能更高，例如linkedList。所以这个标记性只是为了让我们知道我们用什么样的方式去获取数据性能更好。
    　**Cloneable接口**：实现了该接口，就可以使用Object.Clone()方法了。
      **Serializable接口**：实现该序列化接口，表明该类可以被序列化，什么是序列化？简单的说，就是能够从类变成字节流传输，然后还能从字节流变成原来的类。
 - 类属性
```
    public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    // 版本号
    private static final long serialVersionUID = 8683452581122892189L;
    // 缺省容量
    private static final int DEFAULT_CAPACITY = 10;
    // 空对象数组
    private static final Object[] EMPTY_ELEMENTDATA = {};
    // 缺省空对象数组
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    // 元素数组
    transient Object[] elementData;
    // 实际元素大小，默认为0
    private int size;
    // 最大数组容量
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
}
```


----------


## 构造方法 ##
ArrayList有三个构造方法：
    ![](/images/Arraylist_03.gif)
 -无参构造方法
 ```
/**
    * Constructs an empty list with an initial capacity of ten.　　这里就说明了默认会给10的大小，所以说一开始arrayList的容量是10.
    */
　　　　//ArrayList中储存数据的其实就是一个数组，这个数组就是elementData，在123行定义的 private transient Object[] elementData;
　　 public ArrayList() {　　
        super();        //调用父类中的无参构造方法，父类中的是个空的构造方法
        this.elementData = EMPTY_ELEMENTDATA;//EMPTY_ELEMENTDATA：是个空的Object[]， 将elementData初始化，elementData也是个Object[]类型。空的Object[]会给默认大小10，等会会解释什么时候赋值的。
    }
 ```
 
 
 - 有参构造函数一
```
/**
     * Constructs an empty list with the specified initial capacity.
     *
     * @param  initialCapacity  the initial capacity of the list
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public ArrayList(int initialCapacity) {
        super(); //父类中空的构造方法
        if (initialCapacity < 0)    //判断如果自定义大小的容量小于0，则报下面这个非法数据异常
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity]; //将自定义的容量大小当成初始化elementData的大小
    }
```
 - 有参构造方法三(不常用)
 ```
//这个构造方法不常用，举个例子就能明白什么意思
    /*
        Strudent exends Person
         ArrayList<Person>、 Person这里就是泛型
        我还有一个Collection<Student>、由于这个Student继承了Person，那么根据这个构造方法，我就可以把这个Collection<Student>转换为ArrayList<Sudent>这就是这个构造方法的作用 
    */
     public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();    //转换为数组
        size = elementData.length;   //数组中的数据个数
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class) //每个集合的toarray()的实现方法不一样，所以需要判断一下，如果不是Object[].class类型，那么久需要使用ArrayList中的方法去改造一下。
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    }
```

*总结：arrayList的构造方法就做一件事情，就是**初始化一下储存数据的容器**，其实本质上就是一个数组，在其中就叫elementData。*


----------


## 核心方法 ##
 - add()方法（有四个）
 ![](/images/Arraylist_04.gif)
 - boolean add(E)；//默认直接在末尾添加元素
```
/**
     * Appends the specified element to the end of this list.添加一个特定的元素到list的末尾。
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {    
    //确定内部容量是否够了，size是数组中数据的个数，因为要添加一个元素，所以size+1，先判断size+1的这个个数数组能否放得下，就在这个方法中去判断是否数组.length是否够用了。
        ensureCapacityInternal(size + 1);  // Increments modCount!!
     //在数据中正确的位置上放上元素e，并且size++
        elementData[size++] = e;
        return true;
    }
```
分析：**ensureCapacityInternal(xxx)**;　确定内部容量的方法


```
private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
//minCapacity如果大于了实际elementData的长度，那么就说明elementData数组的长度不够用，不够用那么就要增加elementData的length。这里有的同学就会模糊minCapacity到底是什么呢，这里给你们分析一下

/*第一种情况：由于elementData初始化时是空的数组，那么第一次add的时候，minCapacity=size+1；也就minCapacity=1，在上一个方法(确定内部容量ensureCapacityInternal)就会判断出是空的数组，就会给
　　将minCapacity=10，到这一步为止，还没有改变elementData的大小。
　第二种情况：elementData不是空的数组了，那么在add的时候，minCapacity=size+1；也就是minCapacity代表着elementData中增加之后的实际数据个数，拿着它判断elementData的length是否够用，如果length
不够用，那么肯定要扩大容量，不然增加的这个元素就会溢出。
*/


        if (minCapacity - elementData.length > 0)
    //arrayList能自动扩展大小的关键方法就在这里了
            grow(minCapacity);
    }
```
**grow(xxx)**; arrayList核心的方法，能扩展数组大小的真正秘密。
```
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;  //将扩充前的elementData大小给oldCapacity
        int newCapacity = oldCapacity + (oldCapacity >> 1);//newCapacity就是1.5倍的oldCapacity
        if (newCapacity - minCapacity < 0)//这句话就是适应于elementData就空数组的时候，length=0，那么oldCapacity=0，newCapacity=0，所以这个判断成立，在这里就是真正的初始化elementData的大小了，就是为10.前面的工作都是准备工作。
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)//如果newCapacity超过了最大的容量限制，就调用hugeCapacity，也就是将能给的最大值给newCapacity
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
    //新的容量大小已经确定好了，就copy数组，改变容量大小咯。
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
hugeCapacity()
```
//这个就是上面用到的方法，很简单，就是用来赋最大值。
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
//如果minCapacity都大于MAX_ARRAY_SIZE，那么就Integer.MAX_VALUE返回，反之将MAX_ARRAY_SIZE返回。因为maxCapacity是三倍的minCapacity，可能扩充的太大了，就用minCapacity来判断了。
//Integer.MAX_VALUE:2147483647   MAX_ARRAY_SIZE：2147483639  也就是说最大也就能给到第一个数值。还是超过了这个限制，就要溢出了。相当于arraylist给了两层防护。
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```


----------

 - void add(int，E)；在特定位置添加元素，也就是插入元素
 ```
 public void add(int index, E element) {
        rangeCheckForAdd(index);//检查index也就是插入的位置是否合理。

//跟上面的分析一样，具体看上面
        ensureCapacityInternal(size + 1);  // Increments modCount!!
//这个方法就是用来在插入元素之后，要将index之后的元素都往后移一位，
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
//在目标位置上存放元素
        elementData[index] = element;
        size++;//size增加1
    }
 ```
 分析：rangeCheckForAdd(index)　
 ```
 

    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)   //插入的位置肯定不能大于size 和小于0
//如果是，就报这个越界异常
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
/*
public static void arraycopy(Object src,
             int srcPos,
             Object dest,
             int destPos,
             int length)
src：源对象
srcPos：源对象对象的起始位置
dest：目标对象
destPost：目标对象的起始位置
length：从起始位置往后复制的长度。
*/
```
**总结**：
正常情况下会扩容1.5倍，特殊情况下（新扩展数组大小已经达到了最大值）则只取最大值。
当我们调用add方法时，实际上的函数调用如下：
![](/images/Arraylist_05.gif)
举例说明一：
    List<Integer> lists = new ArrayList<Integer>(6);
　　lists.add(8);
说明：初始化lists大小为0，调用的ArrayList()型构造函数，那么在调用lists.add(8)方法时，会经过怎样的步骤呢?
    ![](/images/Arraylist_06.gif)
    *说明*：我们可以看到，在add方法之前开始elementData = {}；调用add方法时会继续调用，直至grow，最后elementData的大小变为10，之后再返回到add函数，把8放在elementData[0]中。
    
举例说明二：
    List<Integer> lists = new ArrayList<Integer>(6);
　　lists.add(8);
　说明：调用的ArrayList(int)型构造函数，那么elementData被初始化为大小为6的Object数组，在调用add(8)方法时，具体的步骤如下:
　  ![](/images/Arraylist_07.gif)
**说明**：我们可以知道，在调用add方法之前，elementData的大小已经为6，之后再进行传递，不会进行扩容处理


----------

## 删除方法 ##

 - remove(int)：通过删除指定位置上的元素
 ```
 public E remove(int index) {
        rangeCheck(index);//检查index的合理性

        modCount++;//这个作用很多，比如用来检测快速失败的一种标志。
        E oldValue = elementData(index);//通过索引直接找到该元素

        int numMoved = size - index - 1;//计算要移动的位数。
        if (numMoved > 0)
//这个方法也已经解释过了，就是用来移动元素的。
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
//将--size上的位置赋值为null，让gc(垃圾回收机制)更快的回收它。
        elementData[--size] = null; // clear to let GC do its work
//返回删除的元素。
        return oldValue;
    }
 ```
 
 

 - remove(Object)：这个方法可以看出来，arrayList是可以存放null值得。
 ```
 //感觉这个不怎么要分析吧，都看得懂，就是通过元素来删除该元素，就依次遍历，如果有这个元素，就将该元素的索引传给fastRemobe(index)，使用这个方法来删除该元素，
//fastRemove(index)方法的内部跟remove(index)的实现几乎一样，这里最主要是知道arrayList可以存储null值
     public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
 ```

 - clear()：将elementData中每个元素都赋值为null，等待垃圾回收将这个给回收掉，所以叫clear
 ```
 public void clear() {
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }
 ```


 - removeAll(collection c)：
 ```
 public boolean removeAll(Collection<?> c) {
         return batchRemove(c, false);//批量删除
     }
 ```

 - batchRemove(xx,xx)：用于两个方法，一个removeAll()：它只清楚指定集合中的元素，retainAll()用来测试两个集合是否有交集。
```
//这个方法，用于两处地方，如果complement为false，则用于removeAll如果为true，则给retainAll()用，retainAll（）是用来检测两个集合是否有交集的。
   private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData; //将原集合，记名为A
        int r = 0, w = 0;   //r用来控制循环，w是记录有多少个交集
        boolean modified = false;  
        try {
            for (; r < size; r++)
//参数中的集合C一次检测集合A中的元素是否有，
                if (c.contains(elementData[r]) == complement)
//有的话，就给集合A
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
//如果contains方法使用过程报异常
            if (r != size) {
//将剩下的元素都赋值给集合A，
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
//这里有两个用途，在removeAll()时，w一直为0，就直接跟clear一样，全是为null。
//retainAll()：没有一个交集返回true，有交集但不全交也返回true，而两个集合相等的时候，返回false，所以不能根据返回值来确认两个集合是否有交集，而是通过原集合的大小是否发生改变来判断，如果原集合中还有元素，则代表有交集，而元集合没有元素了，说明两个集合没有交集。
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
```
 **总结**：：remove函数用户移除指定下标的元素，此时会把指定下标到数组末尾的元素向前移动一个单位，并且会把数组最后一个元素设置为null，这样是为了方便之后将整个数组不被使用时，会被GC，可以作为小的技巧使用。
 
 


----------
## set方法 ##
 ```
 public E set(int index, E element) {
        // 检验索引是否合法
        rangeCheck(index);
        // 旧值
        E oldValue = elementData(index);
        // 赋新值
        elementData[index] = element;
        // 返回旧值
        return oldValue;
    }
 ```
 ## indexOf()方法 ##
 ```
 // 从首开始查找数组里面是否存在指定元素
    public int indexOf(Object o) {
        if (o == null) { // 查找的元素为空
            for (int i = 0; i < size; i++) // 遍历数组，找到第一个为空的元素，返回下标
                if (elementData[i]==null)
                    return i;
        } else { // 查找的元素不为空
            for (int i = 0; i < size; i++) // 遍历数组，找到第一个和指定元素相等的元素，返回下标
                if (o.equals(elementData[i]))
                    return i;
        } 
        // 没有找到，返回空
        return -1;
    } 
 ```
 **说明**：从头开始查找与指定元素相等的元素，注意，是可以查找null元素的，意味着ArrayList中可以存放null元素的。与此函数对应的lastIndexOf，表示从尾部开始查找。
 


----------


 ## get方法 ##
 ```
 

public E get(int index) {
        // 检验索引是否合法
        rangeCheck(index);

        return elementData(index);
    }


 ```
**说明**：get函数会检查索引值是否合法（只检查是否大于size，而没有检查是否小于0），值得注意的是，在get函数中存在element函数，element函数用于返回具体的元素，具体函数如下：
```
E elementData(int index) {
        return (E) elementData[index];
    }
```

大总结:
1）arrayList可以存放null。
2）arrayList本质上就是一个elementData数组。
3）arrayList区别于数组的地方在于能够自动扩展大小，其中关键的方法就是gorw()方法。
4）arrayList中removeAll(collection c)和clear()的区别就是removeAll可以删除批量指定的元素，而clear是全是删除集合中的元素。
5）arrayList由于本质是数组，所以它在数据的查询方面会很快，而在插入删除这些方面，性能下降很多，有移动很多数据才能达到应有的效果
6）arrayList实现了RandomAccess，所以在遍历它的时候推荐使用for循环。