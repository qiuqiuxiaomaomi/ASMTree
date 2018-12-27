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