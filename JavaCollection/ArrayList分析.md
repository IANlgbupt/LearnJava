# ArrayList分析


----------
## ArrayList概述 ##

 - 动态增长和缩减的索引序列，它是基于数组实现的List类
 - 封装了一个动态的object[]数组(即可以存放所有类型数据)，类对象有一个capacity属性，表示它们所封装的Object[]数组的长度。
 - 线程不安全
## ArrayList源码 ##
 - 层次关系：
    ![commit](/images/Arraylist_01.gif)
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




