## Java函数式编程
### 什么是函数式编程
> 参考：http://www.ruanyifeng.com/blog/2012/04/functional_programming.html 

函数式编程可以说是一种编程范式，一种方法论或者一种结构化编程，其主要思想是把运算过程尽量写成一系列嵌套的函数调用。（理解起来很难，算是一种玄学吧...）

### 函数式编程的特点
1、函数是“第一等公民”
<p>所谓"第一等公民"（first class），指的是函数可以赋值给其他变量，也可以作为参数，传入另一个函数，或者作为别的函数的返回值。

2、只用“表达式”，不用“语句”
<p>"表达式"（expression）是一个单纯的运算过程，总是有返回值；"语句"（statement）是执行某种操作，没有返回值。函数式编程要求，只使用表达式，不使用语句。

3、没有“副作用”
<p>所谓"副作用"（side effect），指的是函数内部与外部互动（最典型的情况，就是修改全局变量的值），产生运算以外的其他结果。函数式编程强调没有"副作用"，意味着函数要保持独立，所有功能就是返回一个新的值，没有其他行为，尤其是不得修改外部变量的值。

4、不修改状态
<p>变量往往用来保存"状态"（state），不修改变量，意味着状态不能保存在变量中。

5、引用透明
<p>引用透明（Referential transparency），指的是函数的运行不依赖于外部变量或"状态"，只依赖于输入的参数，任何时候只要参数相同，引用函数所得到的返回值总是相同的。

### 函数式编程的意义
1、代码简洁，开发快速
<p>函数式编程大量使用函数，减少了代码的重复，因此程序比较短，开发速度较快。

2、接近自然语言，易于理解
<p>函数式编程的自由度很高，可以写出很接近自然语言的代码。

3、更方便的代码管理
<p>函数式编程的每一个函数都可以被看做独立单元，很有利于进行单元测试（unit testing）和除错（debugging），以及模块化组合。

4、易于“并发编程”
<p>函数式编程不需要考虑"死锁"（deadlock），因为它不修改变量，所以根本不存在"锁"线程的问题。不必担心一个线程的数据，被另一个线程修改，所以可以很放心地把工作分摊到多个线程，部署"并发编程"（concurrency）。

5、代码的热升级
<p>函数式编程没有副作用，只要保证接口不变，内部实现是外部无关的。所以，可以在运行状态下直接升级代码，不需要重启，也不需要停机。

### Lambda表达式
Java中的函数式编程主要体现在Lambda表达式（Java8支持），使用过高级语言，如：JS、Scala和Kotlin等的朋友应该都很熟悉，这里主要介绍Lambda表达式的一些基本使用和原理。

#### 基本语法
Lambda表达式的语法很简单：<br>
`( 参数 ) -> { 表达式 }` <br>
举例:<br>
1、( ) -> System.out.println("Hello World")
<br>
2、System.out: :println <br>
这种调用叫做方法引用，被引用方法的签名需要与函数接口签名匹配。<br>
3、
```java
(String first, String second) -> {

         if (first.length() < second.length()) return -1;

         else if (first.length() > second.length()) return 1;

         else return 0;

}
```
还有很多用法，这里就不多举例了，大家用起来就熟悉了。

#### 函数式接口
Lambda表达式的基础是函数式接口。所谓函数式接口就是“只有一个抽象方法的接口”。<br>
```java
public interface ILambda {
    void hello(String world); // 只有一个未实现方法
    
    // 其他方法有默认实现
    default void hello2() {
        System.out.println("Hello World");
    }
}
```
Java中还有个函数式接口的注解：`@FunctionalInterface`，但Java并不强制使用。不过，代码中推荐加上这个注解，一方面方便阅读，另一方面编译器会对接口进行检测看是否有多余未实现的方法。
```java
@FunctionalInterface
interface Print<T> {
    void print(T x);
}
```
当然，Java已经为我们提供了很多常用的函数式接口，例如：<br>

![常用函数式接口](https://user-gold-cdn.xitu.io/2018/6/10/163e76a6ffe6ab6d?w=730&h=210&f=png&s=19040)

#### Lambda表达式中的变量和this
1、Lambda表达式中只能引用值不会改变的变量<br>
这里会引出`闭包`的概念，`闭包`理解和使用起来有点难度，感兴趣的朋友自行Google。<br>
2、Lambda表达式中的this指向创建这个Lambda表达式的方法的this参数<br>
```java
public class ThisPointerMain {

    public void doSth() {
        List<Integer> integers = Arrays.asList(2, 4, 6, 8);
        integers.forEach(x -> {
            System.out.println(this.toString()); // 指向ThisPointerMain
            System.out.println(x);
        });
    }

    public void doSth2() {
        List<Integer> integers = Arrays.asList(2, 4, 6, 8);
        integers.forEach(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) {
                System.out.println(this.toString()); // 指向内部类
                System.out.println(integer);
            }
        });
    }

    public static void main(String[] args) {
        ThisPointerMain pointerMain = new ThisPointerMain();
        pointerMain.doSth();
        System.out.println("--------------------------------------------");
        pointerMain.doSth2();
    }

}
```
这个例子运行结果如下：

![this指针指向](https://user-gold-cdn.xitu.io/2018/6/10/163e77ed249db9bc?w=577&h=120&f=png&s=8090)

#### Lambda原理
先给出示例代码：<br>
```java
public class LambdaMain {

    public static void main(String[] args) {

        // 为了导出动态生成的内部类
        System.setProperty("jdk.internal.lambda.dumpProxyClasses", ".");

        // lambda
        printStr("Hello Lambda", (str) -> System.out.println(str));

        // inner class
        printStr("Hello Inner Class", new Printer<String>() {
            @Override
            public void print(String str) {
                System.out.println(str);
            }
        });
    }

    public static void printStr(String str, Printer printer) {
        printer.print(str);
    }

}
```
```java
@FunctionalInterface
public interface Printer<T> {
    void print(T t);
}
```
大胆猜想Lambda表达式也是使用了内部类...那验证一下（下文图片中的文件类名可能和上面的示例对不上，因为是在两台电脑上写的，但例子是相同的）：<br>
**1、看编译产物**
![生成的内部类](https://user-gold-cdn.xitu.io/2018/6/10/163e76deb2cd662a?w=233&h=243&f=png&s=6131)
**只产生了一个内部类的class文件**，难道Lambda表达式不会生成class文件？<br>
<br>
**2、反编译（javap -p）**
![生成静态方法](https://user-gold-cdn.xitu.io/2018/6/10/163e76ef5be7848a?w=674&h=195&f=png&s=21239)
**居然生成了一个私有静态方法！** 那一定在某个地方被调用！ <br>
<br>
**3、再看详细内容（javap -v -p）**
![反编译详情1](https://user-gold-cdn.xitu.io/2018/6/10/163e76f90c36a6cd?w=1049&h=329&f=png&s=33326)
![反编译详情2](https://user-gold-cdn.xitu.io/2018/6/10/163e76fa8bc6a5ff?w=902&h=174&f=png&s=21154)
发现了关键点：<br>
1. **invokestatic和invokedynamic指令**（java7提供的支持动态语言的指令）
2. **LambdaMetafactory.metafactory**（动态生成内部类的关键方法，感兴趣的自行跟踪此方法，就是在运行时生成内部类而不是编译期） 
  <p>每一个invokedynamic指令的实例叫做一个动态调用点(dynamic call site)，动态调用点最开始是未链接状态(unlinked:表示还未指定该调用点要调用的方法)，动态调用点依靠引导方法来链接到具体的方法。引导方法是由编译器生成，在运行期当JVM第一次遇到invokedynamic指令时，会调用引导方法来将invokedynamic指令所指定的名字(方法名，方法签名)和具体的执行代码(目标方法)链接起来，引导方法的返回值永久的决定了调用点的行为。引导方法的返回值类型是java.lang.invoke.CallSite，一个invokedynamic指令关联一个CallSite，将所有的调用委托到CallSite当前的target(MethodHandle)。
  <br>（来自：https://blog.csdn.net/raintungli/article/details/54910152）
  <br>图中，InvokeDynamic #0 就是BootstrapMethods表示#0的位置。
  <br>

`LambdaMetafactory.metafactory`的方法签名如下：<br>
```java
public static CallSite metafactory(
                    // 代表查找上下文与调用者的访问权限，JVM会自动填充
                    MethodHandles.Lookup caller,
                    // 要实现的方法的名字，JVM会自动填充
                    String invokedName,
                    // 调用点期望的方法参数的类型和返回值的类型(方法signature)，JVM会自动填充
                    MethodType invokedType,
                    // 函数对象将要实现的接口方法类型
                    MethodType samMethodType,
                    // 一个直接方法句柄(DirectMethodHandle)，描述在调用时将被执行的具体实现方法 
                    MethodHandle implMethod,
                    // 函数接口方法替换泛型为具体类型后的方法类型，通常和 samMethodType 一样, 不同的情况为泛型
                    MethodType instantiatedMethodType)
```
最后三个参数分别对应了图中的#48、#49和#50。<br>
<br>
**4、查看动态生成的内部类<br>**
开启属性：

```java
System.setProperty("jdk.internal.lambda.dumpProxyClasses", ".");
```

![动态生成的内部类](https://user-gold-cdn.xitu.io/2018/6/10/163e7719785cca22?w=237&h=68&f=png&s=2003)

![反编译动态内部类](https://user-gold-cdn.xitu.io/2018/6/10/163e77289353cece?w=953&h=110&f=png&s=12731)

![动态内部类详情](https://user-gold-cdn.xitu.io/2018/6/10/163e772446a83041?w=1214&h=331&f=png&s=31176)

果然，在这个类里调用了之前生成的静态方法。
<br>所以说，lambda表达式其实是在运行时动态生成了一个内部类。具体怎生成的，得跟踪`LambdaMetafactory.metafactory`方法了，其中关键是`InnerClassLambdaMetafactory#spinInnerClass()`方法。

### Stream类简介
Java中`java.util.stream.Stream`类是java8新增来补充集合类的。<br>
它的使用很丰富，这里简单列几个点：（其实和RxJava很相似）<br>
* 惰性求值方法 & 及早求值方法
* map/reduce/flatMap/filter
* 数据并行化处理

举例：
```java
public static void main(String[] args) {
        // example 1
        System.out.println("-----------------example 1------------------");
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);
        Stream<Integer> numbersStream = numbers.stream();
        int result = numbersStream.map((i) -> i * i)
                .filter((j) -> j % 2 == 0)
                .reduce((x, y) -> x + y)
                .get();
        System.out.println("Result : " + result);


        // example 2
        System.out.println("-----------------example 2------------------");
        List<String> strs = Arrays.asList("A", "B", "C", "D");
        List<String> strsResult = strs.stream()
                .map(String::toLowerCase)
                .collect(Collectors.toList());
        strsResult.forEach(System.out::println);
        strs.forEach(System.out::print);


        // example 3
        System.out.println("-----------------example 3------------------");
        List<Integer> list = Arrays.asList(1, 3, 5, 7, 9, 11);
        int parallelResult = list.stream()
                .parallel()
                .reduce((x, y) -> x * y)
                .get();
        System.out.println("Parallel Result : " + parallelResult);

    }
```

### 比较匿名内部类
最后，比较一下匿名内部类。匿名内部类相对于Lambda表达式有以下不同：<br>
1. 编译器会生成一个内部类文件----影响性能
2. 可以有实例的属性变量----状态
3. 可以含有多个方法
4. 方法体内的this指针指向不同

### 参考资料
[1] https://blog.csdn.net/raintungli/article/details/54910152 <br>
[2] https://luyiisme.github.io/2017/01/21/java8-lambda/ <br>
[3] https://www.cnblogs.com/snowInPluto/p/5981400.html <br>
[4] https://leongfeng.github.io/2016/11/18/java8-function-program-learning/
 <br>
[5] http://ourcoders.com/thread/show/7446/ <br>
[6] http://colobu.com/2016/03/02/Java-Stream/ <br>
[7] http://www.ruanyifeng.com/blog/2012/04/functional_programming.html <br>


