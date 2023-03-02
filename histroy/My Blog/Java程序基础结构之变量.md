
## 1.Java 程序基础结构

```java
/**
 * 可以用来自动创建文档的注释
 */
public class Hello {
    public static void main(String[] args) {
        // 向屏幕输出文本:
        System.out.println("Hello, world!");
        /* 多行注释开始
        注释内容
        注释结束 */
    }
}
```

java 是面向对象的编程语言，因此一个程序的基本单元就是 class, class 是关键字，这里定义的 class 名字就是 Hello。

另外，在 Java 中，对于类名也有一定要求：

- 类名必须以英文字母开头，后接字母，数字和下划线的组合
- 习惯以大写字母开头

上面的 public 是访问修饰符，表示这个 class 是公开的。如果不写 public，也能够编译，但这个类在命令行将无法执行。

在 class 内部可以定义若干的方法：

```java
public class Hello {
    public static void main(String[] args) { // 方法名是main
        // 方法代码...
    } // 方法定义结束
}
```

我们可以在方法中定义一组执行语句，方法内部的代码将会被依此顺序执行。如上代码中方法名是 main,返回值是 void,表示没有任何返回值。

值得注意的是，public 除了可以修饰 class 外，还可以修饰方法。而关键字 static 是另外一个修饰符，他表示静态方法。Java 入口程序规定的方法必须是静态方法，方法名必须是 main,括号内的仓鼠必须是 String 数组。方法名也有命名规则，和 class 差不多，不同的是首字母小写。

在方法内部，语句才是真正执行的代码，Java 的每一句语句都必须以分号结束。而注释也是 Java 中很重要的一环，在 Java 中一共分为三种注释：

1. 单行注释
2. 多行注释
3. 特殊多行注释，需要写在类和方法的定义处，可以用于自动创建文档

```java
/**
 * 这是特殊多行注释可以用来自动创建文档的注释
 *
 * @auther srtian
 */
public class Hello {
    public static void main(String[] args) {
        /*
    这是多行注释
    blablabla...
    这也是注释
    */
    } // 这是单行注释
}
```


## 2.Java 变量和数据类型

Java 中，变量分为两种，基本类型和引用类型（这和 JavaScript 一样）


### 基本类型


#### 变量

在 Java 中，变量必须先被定义后使用,也可以被重新赋值，还可以赋值给其他变量:

```java
public class Hello {
    public static void main(String[] args) {
        int n = 1; // 定义变量n，同时赋值为1
        System.out.println("n = " + n); // 打印n的值

        n = 100; // 变量n赋值为100
        System.out.println("n = " + n); // 打印n的值

        int x = n; // 变量x赋值为n
        System.out.println("x = " + x); // 打印x的值

        x = x + 100; // 变量x赋值为x+100
        System.out.println("x = " + x); // 打印x的值 200
        System.out.println("n = " + n); // 再次打印n的值，100
   }
}
```


#### 基本数据类型

Java 的基本数据类型是可以直接 CPU 进行运算的类型，主要有以下几种：

- 整数类型：byte, short, int, long
- 浮点数类型：float, double
- 字符类型：char
- 布尔类型：boolean

这些基础类型出了其表示的数据不同外，所占的字节数也不同：

```
       ┌───┐
  byte │   │
       └───┘
       ┌───┬───┐
 short │   │   │
       └───┴───┘
       ┌───┬───┬───┬───┐
   int │   │   │   │   │
       └───┴───┴───┴───┘
       ┌───┬───┬───┬───┬───┬───┬───┬───┐
  long │   │   │   │   │   │   │   │   │
       └───┴───┴───┴───┴───┴───┴───┴───┘
       ┌───┬───┬───┬───┐
 float │   │   │   │   │
       └───┴───┴───┴───┘
       ┌───┬───┬───┬───┬───┬───┬───┬───┐
double │   │   │   │   │   │   │   │   │
       └───┴───┴───┴───┴───┴───┴───┴───┘
       ┌───┬───┐
  char │   │   │
       └───┴───┘
```


##### 整数类型

对于整型类型，Java 只定义了带符号的整型，因此，最高位的 bit 表示符号位（0 表示正数，1 表示负数），其范围如下：

```java
byte：-128 ~ 127
short: -32768 ~ 32767
int: -2147483648 ~ 2147483647
long: -9223372036854775808 ~ 9223372036854775807
```

我们可以如下定义整型：

```java
public class Main {
    public static void main(String[] args) {
        int i = 2147483647;
        int i2 = -2147483648;
        int i3 = 2_000_000_000; // 加下划线更容易识别
        int i4 = 0xff0000; // 十六进制表示的16711680
        int i5 = 0b1000000000; // 二进制表示的512
        long l = 9000000000000000000L; // long型的结尾需要加L
    }
}
```


##### 浮点型

浮点型就是小数：

```java
public class Main {
    public static void main(String[] args) {
        float f1 = 3.14f;
        float f2 = 3.14e38f; // 科学计数法表示的3.14x10^38
        double d = 1.79e308;
        double d2 = -1.79e308;
        double d3 = 4.9e-324; // 科学计数法表示的4.9x10^-324
    }
}
```

对于 float 类型需要加上 f 前缀。浮点数可表示的范围非常大，float 类型可最大表示 3.4x1038，而 double 类型可最大表示 1.79x10308


##### 布尔类型

布尔类型 boolean 只有 true 和 false 两个值

```java
boolean b1 = true;
boolean b2 = false;
boolean isGreater = 5 > 3; // 计算结果为true
int age = 12;
boolean isAdult = age >= 18; // 计算结果为false
```

Java 语言对布尔类型的存储并没有做规定，因为理论上存储布尔类型只需要 1 bit，但是通常 JVM 内部会把 boolean 表示为 4 字节整数


##### 字符类型

字符类型 char 表示一个字符。Java 的 char 类型除了可表示标准的 ASCII 外，还可以表示一个 Unicode 字符

```java
public class Main {
    public static void main(String[] args) {
        char a = 'A';
        char zh = '中';
        System.out.println(a);
        System.out.println(zh);
    }
}
```

需要注意的是 char 类型使用单引号，且只有一个字符，要和双引号的字符串类型区分开来


##### 常量

在定义变量的时候，如果在前面加上 final 修饰符，这个变量就变成了常量：

```java
final double a = 3.14
a = 100 // complie error
```

常量在定义时进行初始化后，就不可以再赋值了，这一点和 const 行为是一致的，再次赋值会导致编译错误。

常量最大的作用在于用有意义的变量名来避免魔术数字，根据编程习惯，常量需要全部大写。
