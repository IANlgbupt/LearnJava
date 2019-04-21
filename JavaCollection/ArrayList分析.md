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




