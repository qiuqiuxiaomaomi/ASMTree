# ASMTree
asm技术研究


<pre>
ASM技术简介

         ASM是一个可以操作Java字节码的框架。可以读取、修改class中的字节码。ASM可以直接产生
      二进制class文件，可以在类被加载Java虚拟机之前动态改变类行为，Java class被存储在严
      格格式定义的.class文件里，这些文件拥有足够的元数据来解析所有元素：
         类名称
         方法
         属性
         Java字节码指令
      ASM从类文件中读入信息后，能够改变类行为，分析类信息，甚至能够根据用户要求生成新类。

      对于ASM来说，Java class被描述为一棵树，使用“Vistor”模式遍历整个二进制结构；事件
      驱动的处理方式使得用户只需要关注于其对编程有意义的部分，而不必了解Java类文件格式的
      所有细节：ASM框架提供了默认的"response taker"处理这一切。

      它能方便地生成和改造 Java 代码。著名的框架，如 Hibernate 和 Spring 在底层都用到了 ASM
</pre>

<pre>
 众所周知，Aop 无论概念有多么深奥。它无非就是一个“Propxy模式”。被代理的方法在调用前后作为代理程序可以做一些
 预先和后续的操作。这一点想必读者都能达到一个共识。因此要想实现 Aop 的关键是，如何将我们的代码安插到被调用方法
 的相应位置。
</pre>

<pre>
为什么需要ASM

      动态生成 Java 类与 AOP 密切相关的。AOP 的初衷在于软件设计世界中存在这么一类代码，零散而又耦合：
          1)零散是由于一些公有的功能（诸如著名的 log 例子）分散在所有模块之中；
          2)同时改变 log 功能又会影响到所有的模块。
      出现这样的缺陷，很大程度上是由于传统的 面向对象编程注重以继承关系为代表的“纵向”关系，而对于拥有
      相同功能或者说方面 （Aspect）的模块之间的“横向”关系不能很好地表达

      动态改变 Java 类就是要解决 AOP 的问题，提供一种得到系统支持的可编程的方法，自动化地生成或者增
      强 Java 代码。这种技术已经广泛应用于最新的 Java 框架内，如 Hibernate，Spring 等。 

      利用Proxy代理：
          1）Proxy 是面向接口的，所有使用 Proxy 的对象都必须定义一个接口，而且用这些对象的代码也必
            须是对接口编程的：Proxy 生成的对象是接口一致的而不是对象一致的：例子中 Proxy.newProxyInstance
            生成的是实现 Account接口的对象而不是 AccountImpl的子类。
            这对于软件架构设计，尤其对于既有软件系统是有一定掣肘的。

          2）Proxy 毕竟是通过反射实现的，必须在效率上付出代价：有实验数据表明，调用反射比一般的函数开销至少
            要大 10 倍。而且，从程序实现上可以看出，对 proxy class 的所有方法调用都要通过使用反射的 invoke
            方法。因此，对于性能关键的应用，使用 proxy class 是需要精心考虑的，以避免反射成为整个应用的瓶颈。  

      ASM 能够通过改造既有类，直接生成需要的代码。增强的代码是硬编码在新生成的类文件内部的，没有反射带来性能上的
      付出。同时，ASM 与 Proxy 编程不同，不需要为增强代码而新定义一个接口，生成的代码可以覆盖原来的类，或者是原始
      类的子类。它是一个普通的 Java 类而不是 proxy 类，甚至可以在应用程序的类框架中拥有自己的位置，派生自己的子类。

      相比于其他流行的 Java 字节码操纵工具，ASM 更小更快。ASM 具有类似于 BCEL 或者 SERP 的功能，而只有 33k 大
      小，而后者分别有 350k 和 150k。同时，同样类转换的负载，如果 ASM 是 60% 的话，BCEL 需要 700%，而 SERP 需
      要 1100% 或者更多。

      ASM 已经被广泛应用于一系列 Java 项目：AspectWerkz、AspectJ、BEA WebLogic、IBM AUS、OracleBerkleyDB、
      Oracle TopLink、Terracotta、RIFE、EclipseME、Proactive、Speedo、Fractal、EasyBeans、BeanShell、Groovy、
      Jamaica、CGLIB、dynaop、Cobertura、JDBCPersistence、JiP、SonarJ、Substance L&F、Retrotranslator 等。
      Hibernate 和 Spring 也通过 cglib，另一个更高层一些的自动代码生成工具使用了 ASM。 

      asm字节码增强技术主要是用来反射的时候提升性能的，如果单纯用jdk的反射调用，性能是非常低下的，而使用字节码增
      强技术后反射调用的时间已经基本可以与直接调用相当了
</pre>

ASM的Visitor模式

![](https://i.imgur.com/hFM2RJx.png)

<pre>
流程
      ClassReader:我现在要解析一个复杂结构的类文件，每当我解析好一点东西，都会通知你来处理
      ClassVisitor:你怎么通知我
      ClassReader:当然是回调你的方法，比如说我开始解析一个方法时，我就会回调你的visitMethod(),
                  把相关的数据给你发过去，你就可以处理了
      ClassVisistor:明白，但是当我处理完了，比如修改了字节码，怎么写回去
      ClassReader:你可以创建一个ClassVisitor对象，每当处理完以后，在调用同样的方法，例如
                  visitMethod（），这样ClassWriter就知道你的修改了，它负责写回去。
</pre>

<pre>
字节码解析

      Java 文件：
           public class TestBean {
                  public void halloAop() {
                         System.out.println("Hello Aop");
                  }
           }
     
      Javac之后的class文件
           public class  org.more.test.asm.TestBean extends java.lang.Object{
            
            ##编译器自动生成的默认无参构造函数
			public org.more.test.asm.TestBean();
			  Code:
			   0:   aload_0
			   1:   invokespecial   #8; //Method java/lang/Object."<init>":()V
			   4:   return
			
			public void halloAop();
			  Code:
			   0:   getstatic       #15; //Field java/lang/System.out:Ljava/io/PrintStream;
			   3:   ldc     #21; //String Hello Aop
			   5:   invokevirtual   #23; //Method java/io/PrintStream.println:(Ljava/lang/String;)V
			   8:   return
			}

      ALOAD_0：
          这个指令是LOAD系列指令中的一个，它的意思表示装载当前第 0 个元素到堆栈中。代码上相当于“this”。而这个数据
          元素的类型是一个引用类型。这些指令包含了：ALOAD，ILOAD，LLOAD，FLOAD，DLOAD。区分它们的作用就是针对不
          用数据类型而准备的LOAD指令，此外还有专门负责处理数组的指令 SALOAD。

      invokespecial：
          这个指令是调用系列指令中的一个。其目的是调用对象类的方法。后面需要给上父类的方法完整签名。“#8”的意
          思是 .class 文件常量表中第8个元素。值为：“java/lang/Object."<init>":()V”。结合ALOAD_0。这两个指令
          可以翻译为：“super()”。其含义是调用自己的父类构造方法。

      GETSTATIC：
          这个指令是GET系列指令中的一个其作用是获取静态字段内容到堆栈中。这一系列指令包括了：
          GETFIELD、GETSTATIC。它们分别用于获取动态字段和静态字段。

      IDC：
          这个指令的功能是从常量表中装载一个数据到堆栈中。

      invokevirtual：
          也是一种调用指令，这个指令区别与 invokespecial 的是它是根据引用调用对象类的方法。这里有一篇文章专门讲
          解这两个指令：“http://wensiqun.iteye.com/blog/1125503”。

      RETURN：
          这也是一系列指令中的一个，其目的是方法调用完毕返回：可用的其他指令有：IRETURN，DRETURN，ARETURN等，
          用于表示不同类型参数的返回。
</pre>