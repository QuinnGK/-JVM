## class文件格式

> 为了达到与平台无关的目的，在JVMS中规定byte ordering为大尾方式(大端方式,Big-Endian)，但是在x86平台上数字则以小尾方式(小端方式,Little-Endian)形式存储

## 数据类型
以下两种类型都可用于变量赋值、参数传递、方法返回和运算操作  
* 原始类型(primitive type , 原始值)   
    *  数值型(numeric type)
        *  整数类型(integral type)
            *  byte
            *  short
            *  int
            *  long 
            > 整型默认值都为0，并且全部采用**补码**表示
            *  char
            > char使用16位的**无符号数整数**表示的，指向基本多文种平面的unicode码点
        *  浮点类型(floating-point type)
            *  float 
            *  double
            > 概念上与《IEEE 754》标准中定义的32位单精度和64位双精度的格式与操作一致。**JVM的实现必须支持两种标准的浮点值集合**,**单精度浮点数集合** 和 **双精度浮点数集合** ,可以自由选择实现 **单精度扩展指数集合** 和 **双精度扩展指数集合**  
            > 不同的是**NaN**在JVM中都只用一个值来表示  
            > 把这些浮点数按从小到达排序为 负无穷、可数负数、正负零、可数正数、正无穷   
            > NaN为无序的，并且它和任何数(包括本身)等值比较全部为false
    *  boolean类型  
        > JVM中没有提供任何boolean值的字节码指令，  
        >**java语言表达式所操作的boolean值，在编译之后使用int数据类型来代替**  
        >**JVM直接支持的boolean数组共用byte类型数组的指令。oracle的JVM实现里。boolean数组会被编译为byte数组，每个boolean元素占8位**  
        >true采用1来表示，false采用0表示。编译器将boolean映射为int时也必须如此 
    *  returnAddress类型   
        >  指向的是某个操作码(opcode)的指针，这个类型会被jsr、ret、jsr_w所使用，但是**JDK7中已经不允许使用这几个指令**

* 引用类型(reference type , 引用值)   
    * 类类型(class type)
    > 动态创建的类实例
    * 数组类型(arrary type)
    > 数组实例 
    > 数组类型最外面那一维元素的类型叫做该数组类型的**组建类型**,比如说，int[][][]这个数组来说int[][] 就是这个数组的组件类型，当数组的的组件类型不是数组时，这时就把这种类型称为本数组类型的**元素类型**。  
    **数组的元素类型必须是原生类型、类类型、接口类型之一**
    * 接口类型(interface type)
    > 某个接口的类实例或者数组实例  

    Java虚拟机规范中并没有规定null在虚拟机中应当怎么实现  
    <font color="red">?? 疑问那么class type 与 interface type 有什么区别?</font>
 
## 运行时数据区域

* PC寄存器(program counter|程序计数器)
> JVM是支持多线程同时执行的(JLS第17章)，每条Java线程都有自己的PC寄存器，在任意时刻，一个线程只会执行一个方法(当前方法)，**如果这个方法不是native方法，则PC寄存器就保存Java虚拟机正在执行的字节码指令的地址，如果这个方法是个native方法则PC寄存器的值为undefined**。  
PC寄存器的容量至少应当能保存一个returnAddress类型的数据或者一个与平台相关的本地指针的值  
<font color="red">?? 那么一个returnAddress占几个字节，平台相关的本地指针是什么？</font>

* Java虚拟机栈
> 每一条线程都有自己的Java虚拟机栈，这个栈与线程同时创建，用于存储栈帧(Frame)，除了pop|push之外，Java虚拟机栈不会受到其他因素影响，所以栈帧可以在堆中分配，Java虚拟机所使用的内存不需要保证为连续的。  
Java虚拟机栈可以为固定大小，也可以动态扩展或者收缩。  
**应当提供调节Java虚拟机栈的初始容量的手段，或者调节最大或者最小容量手段。**  
当线程请求分配栈容量超过Java虚拟机允许的最大容量会抛出StackOverflowError异常。  
当Java虚拟机栈动态扩展时，无法申请到足够的内存或者在创建新线程时没有足够的内存去创建对应的虚拟机栈时，会抛出OutOfMemoryError异常。  
<font color="red">?? 栈 堆 与 java栈 java堆的区别是什么，为什么说栈帧可以在堆中分配?</font>  
