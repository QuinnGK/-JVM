# JVM参数
*  -XX:+TraceClassLoading(类加载信息)
*  -XX:+TraceClassUnloading(类卸载信息)
*  -Xverify:non(关闭类验证)  
# 字节码指令
*   newarray(创建一个数组)
# 运行期 


## 1.加载(Loading)  
> JVM规范并没有严格约束类加载的时机,统一由虚拟机实现方自由把握
> 加载分为三个小步骤
>>1.通过一个类的全限定名来获取定义此类的二进制字节流。  
>>2.二进制字节流就按照虚拟机所需的格式存储在方法区之中，方法区中的数据存储格式由虚拟机实现自行定义，虚拟机规范未规定此区域的具体数据结构  
>>3.内存中实例化一个java.lang.Class类的对象（并没有明确规定是在Java堆中，对于HotSpot虚拟机而言，Class对象比较特殊，它虽然是对象，但是存放在方法区里面），这个对象将作为程序访问方法区中的这些类型数据的外部接口  

> 由1可知,类加载器要的是一个二进制的字节流所以那么这个字节流可以从好多种方式获取
>> 例如从数据库、网络、jar、zip包、动态代理技术，在java.lang.reflect.Proxy中，就是用了ProxyGenerator.generateProxyClass来为特定接口生成形式为"*$Proxy"的代理类的二进制字节流(**后期会在设计模式笔记 Proxy模式中详细说明 JDK代理这类**)
> **重点：加载一个类可能尚未完成时这个类的连接阶段就开始了,并不是要等到加载全部完成才开始连接阶段**


## 2.连接(Linking)
* **验证(Verification)**
>>1.文件格式验证  
验证字节流是否符合Class文件格式的规范  
>>2.元数据验证  
对字节码描述的信息进行语义分析，以保证其描述的信息符合Java语言规范的要求  
>>3.字节码验证  
通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的,这个阶段将对类的方法体进行校验分析，保证被校验类的方法在运行时不会做出危害虚拟机安全的事件    
>>JDK1.6之后在字节码方法表的Code属性的属性表中加入StackMapTable属性用于做类型验证(**详细请移步字节码结构**)   
>>类型推导方式进行校验与类型检查验证(stackMapTable)这两种实现方式   
>>4.符号引用验证   
**解析阶段中发生**。符号引用验证可以看做是对类自身以外（常量池中的各种符号引用）的信息进行匹配性校验

>>详细信息请移步官方文档[格式检查](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.8)、[验证class文件](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.10)、[虚拟机代码的约束](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.9)

* **准备(Preparation)**
>对静态变量在方法区中分配空间,并且赋默认值。  

数据类型|默认值
:----:|:---:
  int |  0  
 long |  0
 short|  0
 char |  0
 byte |  0
 float|  0
 double| 0
 boolean|  0
 reference| 0

> **重点**：在准备阶段,如果类字段的字段属性表中存在ConstantValue属性，那在准备阶段变量value就会被初始化为ConstantValue属性所指定的值,
这里需要在强调一下，目前Sun Javac编译器的选择是：如果同时使用final和static来修饰一个变量（按照习惯，这里称“常量”更贴切），并且这个变量的数据类型是基本类型或者java.lang.String的话，就会在对应的字段的属性表中加入ConstantValue属性来进行初始化，**然而虚拟机规范中并没有强制要求字段访问标志必须有ACC_FINAL**,只要包含ACC_STATIC,所以这些都是javac做的事。
(**ConstantValue请移步字节码结构**)

* **解析(Resolution)**
> **解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程**
>>符号引用（Symbolic References）：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。   
>>直接引用（Direct References）：直接引用可以是直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄。
>>当遇到anewarray、checkcast、getfield、getstatic、instanceof、invokedynamic、invokeinterface、invokespecial、invokestatic、invokevirtual、ldc、ldc_w、multianewarray、new、putfield和putstatic这16个用于操作符号引用的字节码指令之前，先对它们所使用的符号引用进行解析   

>>1.接口或者类的解析
>>>根据符号引用保存的全限定名，传入当前类的类加载中。让当前类的类加载器去加载这个类。然后做一些元数据验证、字节码验证，以及触发被加载类的其他相关类的加载(暂时没有想到什么递归的去加载的，如果AextendB extend C ,那么我在新建A时去调用 C 的方法，A是不做初始化的，C才会做初始化。但是问题来了。A不做初始化。不执行<clinit> 就不会触发上述指令吧,是什么时候加载到C的呢。还是 和虚拟方法吧有关。需要在研究一下),最后再做符号引用验证。如果为数组。则去加载数组元素类型。并有虚拟机生成一个代表此数组维度和元素的数组对象

## 3.初始化(Initialization)
> 初始化时调用<clinit>()方法对类进行初始化，通常就是从上至下的顺序对代码中的静态变量赋值以及执行静态代码块
> ### JVM规范中规定了五种情况下需要执行初始化操作,并且以下五种方式都算作:**主动引用**  
>>1.遇到**new**(new一个对象),**getstatic**(获得一个类的静态变量),**putstatic**(对一个类的静态变量赋值),**invokestatic**(调用一个类的静态方法)这四个字节码指令  
 >>2.使用java.lang.reflect包的方法**对类进行反射调用**时  
 >>3.当初始化一个类时**其父类先触发初始化**  
 >>4.虚拟机启动时**用户主动指定的主类**，例如main()的类  
 >>5.当使用JDK 1.7的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化  
 >### **被动引用**
 >>1.除了上述主动引用的方式意外所有的引用方式都称为被动引用  
 >>2.这里要详细的说下如果是一个接口被初始化的时候,其父接口则不需要初始化,这一点是接口与类不同的地方  
 >>3.数组是由JVM自动创建的,ClassLoad类注解中提到,原文如下"objects for array classes are not created by class loaders, but are created automatically as required by the Java runtime.The class loader for an array class, as returned by is the same as the class loader for its element type; if the element type is a primitive type, then the array class has no class loader."   
 >>4.如过A类引用了B类的一个常量,并且这个常量是一个不用在运行期就能确定的值,那么在编译期,就会将这个常量放入A类的常量池,自然就不会加载B类,直接运行A类即可(简单的说就是你将编译后的B.class文件删除后一样可以执行A.class文件)  
 >>5.如果一个类中A类继承B类,A类中有个静态变量a,如果C类中通过B.a去引用A类中的静态变量时,B类不做初始化.因为a是A的静态变量所以,只会初始化A类。B类是否做加载和验证在虚拟机规范中并未明确规定。
## 4.使用(Using)

## 5.卸载(Unloading)
