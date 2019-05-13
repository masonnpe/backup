java的平台无关性怎么实现的？    java文件编译成字节码class文件   jvm 再执行成对应不同平台的机器码

jvm怎么加载class文件？   classloader 负责加载符合格式的 到内存  execute engine  对命令解析

什么是反射    class.forname  newinstance 得到对象

什么是classloader？    loadclass方法 class 变成二进制流加载     

> bootstrapclassloader  记载java.
>
> extclassloader 记载javax 扩展的
>
> appclassloader  记载程序所在目录
>
> 自定义classloader



双亲委派 ：先看自己有没有加载   判断parent是否为空    都没有   就去调用findclass 尝试去加载

​	可以防止多份同样字节码的加载



loadclass 和 class.forname区别

还没链接      已经初始化的



为什么递归会导致stackoverflow？  每次调用都会压入一个栈帧





堆和栈区别

栈自动释放  堆要gc

堆空间大

堆碎片多



```
String a=new String("a");
a.intern();
String b="a";
System.out.println(a==b);//false

String c=new String("a")+new String("a");
c.intern();
String d="aa";
System.out.println(c==d);//true
```