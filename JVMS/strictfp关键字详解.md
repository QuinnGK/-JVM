## strictfp关键字

>关键字strictfp,即 strict float point (精确浮点)。strictfp 关键字可应用于类、接口或方法。
使用 strictfp 关键字声明一个方法时，该方法中所有的float和double表达式都严格遵守FP-strict的限制,符合IEEE-754规范。  
当对一个类或接口使用 strictfp 关键字时，该类中的所有代码，包括嵌套类型中的初始设定值和代码，都将严格地进行计算。  
严格约束意味着所有表达式的结果都必须是 IEEE 754 算法对操作数预期的结果，以单精度和双精度格式表示。    
如果你想让你的浮点运算更加精确，而且不会因为不同的硬件平台所执行的结果不一致的话，可以用关键字strictfp.
以上为wiki中对这个关键字的介绍。  

但是这个关键字并没有出现在Class的ACC_FLAG中  

以下为类的ACC_FLAG表  
![CLASS_ACC_FLAG](https://raw.githubusercontent.com/QuinnGK/JVM/master/JVMS/image/Class%20ACC_FLAG.jpeg)

可以看到在上面并没有发现ACC_STRICT。而是在方法的ACC_FLAG中出现  

以下为方法的ACC_FLAG表
![METHOD_ACC_FLAG](https://raw.githubusercontent.com/QuinnGK/JVM/master/JVMS/image/Method_ACC_FLAG.jpeg)

再来看字节码 
## strictfp修饰的类
```java
public strictfp class TestStrictfp {

    public void test(){

    }
}

//byteCode
public class cn/infisa/chronic2/base/TestStrictfp {
  // compiled from: TestStrictfp.java
  // access flags 0x801
  public strictfp <init>()V
   L0
    LINENUMBER 8 L0
    ALOAD 0
    INVOKESPECIAL java/lang/Object.<init> ()V
    RETURN
   L1
    LOCALVARIABLE this Lcn/infisa/chronic2/base/TestStrictfp; L0 L1 0
    MAXSTACK = 1
    MAXLOCALS = 1

  // access flags 0x801
  public strictfp test()V
   L0
    LINENUMBER 12 L0
    RETURN
   L1
    LOCALVARIABLE this Lcn/infisa/chronic2/base/TestStrictfp; L0 L1 0
    MAXSTACK = 0
    MAXLOCALS = 1
}
```

可以看到类上并没吹昂strictfp关键字。而是在每个方法中出现。看来是javac干了这件事

## strictfp 作用在方法上
```java 
public  class TestStrictfp {

    public strictfp void test(){

    }
}

// class version 52.0 (52)
// access flags 0x21
public class cn/infisa/chronic2/base/TestStrictfp {

  // compiled from: TestStrictfp.java

  // access flags 0x1
  public <init>()V
   L0
    LINENUMBER 8 L0
    ALOAD 0
    INVOKESPECIAL java/lang/Object.<init> ()V
    RETURN
   L1
    LOCALVARIABLE this Lcn/infisa/chronic2/base/TestStrictfp; L0 L1 0
    MAXSTACK = 1
    MAXLOCALS = 1

  // access flags 0x801
  public strictfp test()V
   L0
    LINENUMBER 12 L0
    RETURN
   L1
    LOCALVARIABLE this Lcn/infisa/chronic2/base/TestStrictfp; L0 L1 0
    MAXSTACK = 0
    MAXLOCALS = 1
}
```
可以看到只有test一个方法有strictfp修饰符