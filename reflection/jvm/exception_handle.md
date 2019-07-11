## 异常处理

### 定义

jvm的异常处理，本质上是对一段代码进行监控，并实现执行流的**转移**

* 从方法正常执行路径转移到异常执行路径


### 处理方式

异常处理表检查机制: 抛出异常后，去检查ExceptionTable并实现跳转

* 监控范围： from -> to bytecode index
* 异常类型： type
	* Exception等
	* any: 异常执行路径抛出的异常
* 目标位置: tartget -> bytecode index, handler exception type


### 实例

```
/**
 * 查看jvm的异常处理机制
 * 1 异常处理表：exception table
 * 2 目标跳转
 */
public class ExceptionShower {

    private int tryBlock;

    private int catchBlock;

    private int finallyBlock;

    private int methodExit;


    public void showException(){
        try {
            tryBlock = 0;
        }catch (Exception e){
            catchBlock = 1;
        }finally {
            finallyBlock = 2;
        }

        methodExit = 3;
    }

}

```

**javap -c 字节码反汇编：**

```
public void showException();
    Code:
       0: aload_0
       1: iconst_0
       2: putfield      #2                  // Field tryBlock:I
       5: aload_0
       6: iconst_2
       7: putfield      #3                  // Field finallyBlock:I
      10: goto          35
      13: astore_1
      14: aload_0
      15: iconst_1
      16: putfield      #5                  // Field catchBlock:I
      19: aload_0
      20: iconst_2
      21: putfield      #3                  // Field finallyBlock:I
      24: goto          35
      27: astore_2
      28: aload_0
      29: iconst_2
      30: putfield      #3                  // Field finallyBlock:I
      33: aload_2
      34: athrow
      35: aload_0
      36: iconst_3
      37: putfield      #6                  // Field methodExit:I
      40: return
    Exception table:
       from    to  target type
           0     5    13   Class java/lang/Exception
           0     5    27   any
          13    19    27   any
```

**解读：**

* try, catch的出口会添加finally字节码
* 异常执行路径下抛出的异常类型为any 


---

### 为什么异常处理会比较消耗性能

构造异常(Throwable)时，需要填充栈调用轨迹(fillInStackTrace)，该方法比较消耗性能

```

 	/**
     * Fills in the execution stack trace. This method records within this
     *
     * @return  a reference to this {@code Throwable} instance.
     */
	public synchronized Throwable fillInStackTrace() {
        if (stackTrace != null ||
            backtrace != null /* Out of protocol state */ ) {
            fillInStackTrace(0);
            stackTrace = UNASSIGNED_STACK;
        }
        return this;
    }
    
```

该方法会查看每个栈帧并记录调用信息，例如方法名，所在类等


### 补充

* 尽量用try-catch-resources机制处理资源关闭，避免二次抛出异常后无法追溯源异常
* 如果几个异常的处理方式一致，可以在一个catch内捕获处理

```
 public void catchMultipleException(){
        try {
            List<Integer> testList = new ArrayList<>();
            System.out.println(testList.get(0));
        }catch (ArrayIndexOutOfBoundsException | NullPointerException e){
            e.printStackTrace();
        }
    }

```



