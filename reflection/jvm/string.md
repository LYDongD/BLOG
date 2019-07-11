## 关于字符串的两个问题

先来看一段代码

```
 public static void strTest01(){
        String s = "11";
        String s2 = new String("11");
        System.out.println(s == s2);
 }

```

对于这段代码，产生以下几个问题：

1. 字面量11保存在哪里？
2. 运行结果是什么，原因是什么?

对这段代码进行编译后，使用命令：

javap -v xx.class 反编译，查看方法执行指令和其他类信息

```

#常量池
Constant pool:
   #1 = Methodref          #9.#30         // java/lang/Object."<init>":()V
   #2 = String             #31            // 11
   #3 = Class              #32            // java/lang/String
   ...
   #30 = NameAndType        #10:#11        // "<init>":()V
   #31 = Utf8               11
   ...

#方法指令
public static void strTest01();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=2, args_size=0
         0: ldc           #2                  // String 11
         2: astore_0
         3: new           #3                  // class java/lang/String
         6: dup
         7: ldc           #2                  // String 11
         9: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
        12: astore_1
        13: getstatic     #5                  // Field java/lang/System.out:Ljava/io/PrintStream;
        16: aload_0
        17: aload_1
        18: if_acmpne     25
        21: iconst_1
        22: goto          26
        25: iconst_0
        26: invokevirtual #6                  // Method java/io/PrintStream.println:(Z)V
        29: return
```

从常量池可以看出，字面量本质上常量，会被加载到常量池

从方法执行指令可以看出，两行语句的指令不一样：

String s = "11" 分解为2个指令，从常量池索引#2查找常量并加载，写入局部变量表

String s2 = new String("11") 则分解成5个指令，最终执行invokespecial初始化对象String

**s保存的是字面量，s2保存的是字符串String对象的地址，因此2个变量的值不同，执行结果为false**


----


既然说，字面量是保存在常量池中，那么一下代码片段的执行结果又是怎样的呢？

```

public static void strTest02(){
        String s = "11";
        String s2 = "1" + "1";
        System.out.println(s == s2);
    }

```

常量池中，是否会同时保存字面量1和11？

对字节码反编译，结果如下：

```

#常量池
Constant pool:
   #1 = Methodref          #9.#31         // java/lang/Object."<init>":()V
   #2 = String             #32            // 11
   #3 = Class              #33            // java/lang/String
   ...

#方法执行指令
public static void strTest02();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=2, args_size=0
         0: ldc           #2                  // String 11
         2: astore_0
         3: ldc           #2                  // String 11
         5: astore_1
         6: getstatic     #5                  // Field java/lang/System.out:Ljava/io/PrintStream;
         9: aload_0
        10: aload_1
        11: if_acmpne     18
        14: iconst_1
        15: goto          19
        18: iconst_0
        19: invokevirtual #6                  // Method java/io/PrintStream.println:(Z)V
        22: return


```

从常量池来看，只有一个11一个常量，说明“1” + “1” 在**编译阶段**就合并为“11”

**从方法执行指令来看，s和s2对应的指令是完全一样的，都保存了常量11, 所以结果返回true**


