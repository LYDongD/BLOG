## JVM 方法调用及虚方法优化机制

### jvm方法调用对应指令

* invokespecial //super，构造方法和私有实例方法调用
* invokeinterface // 接口方法
* invokestatic //静态方法
* invokevirtual //非私有的实例方法
* invokedynamic //动态方法(不在本文讨论范围)

**分类**：

* 虚方法，类解析时将符号引用解析成类方法表的索引
	* invokeinterface
	* invokevirtual

<font color=red> 虚方法除了final方法外，都需要动态绑定，即在运行时通过调用者的动态类及方法表索引获取目标方法 </font>

* 非虚方法，类解析时将符号引用解析成目标方法的引用
	* invokespecial
	* invokestatic

<font color=red> 非虚方法，静态绑定，解析时就能获取目标方法的引用，可直接调用 </font>
	


#### 举例：

```

//公司
public class Company {

    public double giveSalary(double salary, Person person){
        return salary;
    }
}

//员工
public interface Person {

    public boolean isSmart();
}

//it公司给员工发公司
public class ItCompany extends Company{

    public double giveSalary(double salary, Person person){
		
		  //接口方法，invokeinterface
        if (person.isSmart()){
        	  //静态方法，invokestatic
            double smartIndex = smartIndex();
            //私有实例方法，invokespecial
            printSmartIndex(smartIndex);
            return salary * smartIndex;
        }

		 //向上引用调用，invokespecial
        return super.giveSalary(salary, person);
    }

    private void printSmartIndex(double smartIndex){
        System.out.println(smartIndex);
    }

    public static double smartIndex(){
    	 //普通实例方法调用， invokevirtual
        return new Random().nextDouble() + 1;
    }
}


```

javap -c 反汇编：

```
public class com.liam.demo.jvm.methodCall.ItCompany extends com.liam.demo.jvm.methodCall.Company {
  public com.liam.demo.jvm.methodCall.ItCompany();
    Code:

com.liam.demo.jvm.methodCall.ItCompany
       0: aload_0
       1: invokespecial #1                  // Method com/liam/demo/jvm/methodCall/Company."<init>":()V
       4: return

  public double giveSalary(double, com.liam.demo.jvm.methodCall.Person);
    Code:
       0: aload_3
       1: invokeinterface #2,  1            // InterfaceMethod com/liam/demo/jvm/methodCall/Person.isSmart:()Z
       6: ifeq          25
       9: invokestatic  #3                  // Method smartIndex:()D
      12: dstore        4
      14: aload_0
      15: dload         4
      17: invokespecial #4                  // Method printSmartIndex:(D)V
      20: dload_1
      21: dload         4
      23: dmul
      24: dreturn
      25: aload_0
      26: dload_1
      27: aload_3
      28: invokespecial #5                  // Method com/liam/demo/jvm/methodCall/Company.giveSalary:(DLcom/liam/demo/jvm/methodCall/Person;)D
      31: dreturn

  public static double smartIndex();
    Code:
       0: new           #8                  // class java/util/Random
       3: dup
       4: invokespecial #9                  // Method java/util/Random."<init>":()V
       7: invokevirtual #10                 // Method java/util/Random.nextDouble:()D
      10: dconst_1
      11: dadd
      12: dreturn
}

```

---

### 虚方法动态绑定优化机制

#### 思路
**避免重复读方法表，缓存动态类型的目标方法地址**

**即内联缓存：**

* 单态：仅缓存一种动态类型
* 多态：缓存有限数量动态类型
* 超多态：某个临界值之上的状态

#### jvm的实现

单态内联缓存 + 超多态劣化

即仅缓存一个动态类型的目标方法，如果动态类型变化，则劣化成去方法表查找

**好处：节省内存，避免ABAB调用导致一直写缓存，没有起到性能提升的作用**

#### 举例

1. 定义虚方法：exit()
2. 动态类型1连续调用若干次虚方法，触发单态内联缓存
3. 动态类型2连续调用若干次虚方法，触发超多态劣化

期望：动态类型2调用方法的时间 > 动态类型1调用方法的时间

```

public abstract class Passenger {

    public abstract void exit();

    static class ChinesePassenger extends Passenger {

        @Override
        public void exit() {
            //System.out.println("走中国通道");
        }

        public void buy() {
           // System.out.println("买买买");
        }
    }

    static class ForeignerPassenger extends Passenger {

        @Override
        public void exit() {
            //System.out.println("走外国通道");
        }
    }


    /*
     *  java -XX:CompileCommand='dontinline,*.exit' com/liam/demo/jvm/virtualMethodOptimize/Passenger
     *   禁止使用方法内联的情况下, 虚方法exit()被单态内联优化，为ChinesePassenger动态类创建单台缓存，当类型变更
     *   为ForeignerPassenger，退化为超多太劣化，从该类的方法表中取找exit的目标方法，不会更新缓存
     *
     *   因此执行为ForeignerPassenger目标方法所花费的时间更多
     *
     */
    public static void main(String args[]) {
        Passenger chinesePassenger = new ChinesePassenger();
        Passenger foreignerPassenger = new ForeignerPassenger();

        //前1亿次执行动态类chinesePassenger的目标方法，后1亿次执行动态类foreignerPassenger的目标方法，比较执行时间
        long begin = System.currentTimeMillis();
        for (int i = 1; i <= 200000000; i++){
            if (i % 100000000 == 0){
                long tmp = System.currentTimeMillis();
                System.out.println(tmp - begin);
                begin = tmp;
            }
            Passenger passenger = i < 100000000 ? chinesePassenger : foreignerPassenger;
            passenger.exit();
        }
    }

```

