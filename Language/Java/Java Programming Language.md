
第一段Java代码：
```Java
public class Main {  
	public static void main(String[] args) {
		System.out.printf("Hello and welcome!");    
		for (int i = 1; i <= 5; i++) {  
			System.out.println("i = " + i);  
		}
	}
}
```


# Java 基础语法

## 1. 变量和类型

### 1.1 基本数据类型

- 整数类型：byte，short，int，long
- 浮点数类型：float，double
- 字符类型：char
- 布尔类型：boolean

#### 基本数据类型包装类

引入包装类：以对象的方式来操作基本类型的数据，解决一些特定的问题。**基本类型都有相对应的包装器类型**。包装器类型关键字有：Byte、Short、Character、Integer、Long、Boolean、Float、Double。

#### 基本数据类型缓存池

- `new Integer(18)` 每次都会新建一个对象;
- `Integer.valueOf(18)` 会使⽤用缓存池中的对象，多次调用只会取同⼀一个对象的引用。

如 Integer 类内部中内置了 256 个 Integer 类型的缓存数据，当使用的数据范围在 -128~127 之间时，会直接返回常量池中数据的引用，而不是创建对象，超过这个范围时会创建新的对象。


`valueOf` 方法的源码：
```java
public static Integer valueOf(int i) {
    if (i >=IntegerCache.low && i <=IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

`IntegerCache` 静态内部类源码：
```java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue = 
			        sun.misc.VM
			        .getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert Integer.IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```


### 1.2 引用类型

除了上述基本类型的变量，剩下的都是引用类型。例如，引用类型最常用的就是 `String` 字符串，数组是引用类型：

```Java
int[] a = new int[5];  
int[] b = new int[] {1, 2, 3, 4};  
int[] c = {1, 2, 3};
```

#### 数组

数组的声明方式分两种：
```java
int[] anArray;             // 推荐采用第一种声明格式，可读性好
                           // ArrayList 的源码中就用了第一种方式
int anOtherArray[];

int[] anArray = new int[] {1, 2, 3, 4, 5};        // 使用了new关键字
                                                  // 说明数组是一个对象
```

List 封装了很多常用的方法，方便我们对集合进行一些操作，而如果直接操作数组的话，有很多不便，因为数组本身没有提供这些封装好的操作，所以有时候我们需要把数组转成 List。

```java
int[] anArray = new int[] {1, 2, 3, 4, 5};

// 方法一
List<Integer> aList = new ArrayList<>();
for (int element : anArray) {
    aList.add(element);
}

// 方法二
List<Integer> aList = Arrays.asList(anArray);    // 错误，Arrays.asList 的参数需要是 Integer 数组
```

#### String

源码：
```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    
	private final char value[];      // Java9之前，用char存值
	
    @Stable
    private final byte[] value;      // Java9之后改用byte，减少内存，增加编码检测开销
                                     // 被final修饰，说明不可变
    private final byte coder;        // 区别编码方式
    private int hash;                // 每一个字符串都会有一个 hash 值
    
}
```
* String类是 final 的，意味着它不能被子类继承。
* String 类实现了 Serializable 接口，意味着它可以序列化。
* String 类实现了 Comparable 接口，意味着最好不要用 `==` 来比较两个字符串是否相等，而应该用 `compareTo()` 方法去比较。

常用方法：
- `substring(int beginIndex, int endIndex)`：截取字符串
- `indexOf()`：查找一个子字符串在原字符串中第一次出现的位置，并返回该位置的索引。
- `length()`：返回字符串长度。
- `isEmpty()`：用于判断字符串是否为空。
- `charAt()` ：用于返回指定索引处的字符。
- `trim()` ：用于去除字符串两侧的空白字符。
- `split()`：分割字符串
- `intern()`：复制字符串


##### String不可变性

- 保证 String 对象的安全性，避免被篡改。
- 保证哈希值不会频繁变更。
- 可以实现字符串常量池，Java 会将相同内容的字符串存储在字符串常量池中。这样，具有相同内容的字符串变量可以指向同一个 String 对象，节省内存空间。

由于字符串的不可变性，String 类的一些方法实现**最终都返回了新的字符串对象**。


##### String字符串常量池

由于字符串的使用频率高，所以 JVM 为了提高性能和减少内存开销，在创建字符串对象的时候进行了一些优化，特意为字符串在堆中开辟了一块空间，作为**字符串常量池**。使用双引号声明的字符串对象会保存在字符串常量池中。

```java
String s = new String("abc");


String s1 = new String("abc");  
String s2 = new String("abc");  
System.out.println(s1 == s2);    // false
```
上述代码运行行为：
1. JVM 先在字符串常量池中查找有没有 `"abc"` 这个字符串对象。
2. 如果有，直接在堆中创建一个 `"abc"` 的字符串对象，然后将堆中这个 `"abc"` 的对象地址返回赋值给变量 s。
3. 如果没有，先在字符串常量池中创建一个 `"abc"` 的字符串对象，然后再在堆中创建一个 `"abc"` 的字符串对象，然后将堆中这个 `"abc"` 的字符串对象地址返回赋值给变量 s。

```java
String s = "abcd";


String s3 = "abcd";  
String s4 = "abcd";  
System.out.println(s3 == s4);   // true
```
上述代码运行行为：
1. JVM 会先在字符串常量池中查找有没有 `"abcd"` 这个字符串对象。
2. 如果有，直接将字符串常量池中这个 `"abcd"` 的对象地址返回，赋给变量 s。
3. 如果没有，在字符串常量池中创建 `"abcd"` 这个对象，然后将其地址返回，赋给变量 s。


```java
String s1 = new String("bcdef");
String s2 = s1.intern();
System.out.println(s1 == s2);   // false

String s7 = "bcde";  
String s8 = s7.intern();  
System.out.println(s7 == s8);   // true

String s9 = new String("abc") + new String("def");  
String s10 = s9.intern();  
System.out.println(s9 == s10);  // true
```
上述代码运行行为：
1. 字符串常量池中会先创建一个 `bcdef` 的对象，然后堆中会再创建一个 `bcdef` 的对象，s1 引用的是堆中的对象。
2. s2 对 s1 执行 `intern()` 方法，该方法会从字符串常量池中查找 `bcdef` 这个字符串是否存在，此时是存在的，所以 s2 引用的是字符串常量池中的对象。
3. s1 引用堆中对象，s2 引用字符串常量池中对象，因此两者不相等。
4. s7 和 s8 都指向字符串常量池中的对象。
5. s9 在字符串常量池中创建两个对象，一个是 `abc` ，一个是 `def` ，然后在堆中会创建两个匿名对象 `abc` 和 `def` ，最后还有一个 `abcdef` 的对象，s9 引用的是堆中 `abcdef` 这个对象。
6. s10 对 s9 执行 `intern()` 方法，该方法会从字符串常量池中查找 `abcdef` 这个对象是否存在，此时不存在的，但堆中已经存在了，所以**字符串常量池中保存的是堆中这个 `abcdef` 对象的引用**，也就是说，s9 和 s10 的引用地址是相同的。
7. `new String("abc") + new String("def")` 会创建一个 StringBuilder 对象，并将 `abc` 和  `def`  追加到其中，然后调用 StringBuilder 对象的 `toString()` 方法，将其转换为一个新的字符串对象，内容为 `abcdef` ，这个新的字符串对象存储在堆上。


##### StringBuilder和StringBuffer

由于字符串是不可变的，所以当遇到字符串拼接（尤其是使用+号操作符）的时会产生太多 String 对象，为了缓解内存压力，Java设计了StringBuffer类和StringBuilder类。

StringBuffer：
```java
public final class StringBuffer extends AbstractStringBuilder 
		implements Serializable, CharSequence 
{

    public StringBuffer() {
        super(16);
    }
    
    public synchronized StringBuffer append(String str) {
        super.append(str);
        return this;
    }

    public synchronized String toString() {
        return new String(value, 0, count);
    }

    // ...
}
```

StringBuilder：
```java
public final class StringBuilder extends AbstractStringBuilder 
		implements java.io.Serializable, CharSequence
{
    // ...

    public StringBuilder append(String str) {
        super.append(str);
        return this;
    }

    public String toString() {
        // Create a copy, don't share the array
        return new String(value, 0, count);
    }

    // ...
}
```

两者唯一的不同是：StringBuilder 方法没有加 synchronized，StringBuffer 方法加 synchronized。
这是由于 StringBuffer 考虑到多线程环境下的安全问题，却因此执行效率会比较低。

实际开发中，StringBuilder 的使用频率也是远高于 StringBuffer，甚至可以这么说，**StringBuilder 完全取代了 StringBuffer**。


```java
// 使用 +
String result = "";
for (int i = 0; i < 100000; i++) {
    result += "六六六";
}

// 使用append
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 100000; i++) {
    sb.append("六六六");
}
```
上述代码性能差距巨大，原因：循环体内如果用 + 号操作符的话，就会产生大量的 StringBuilder 对象，不仅占用了更多的内存空间，还会让 JVM 不停的进行垃圾回收，从而降低了程序的性能。


##### 字符串相等判断

-  `==` 操作符用于比较两个对象的地址是否相等，要求必须是同一个对象。
- `.equals()` 方法用于比较两个对象的内容是否相等。



### 1.3 变量

变量按照作用域的范围可分为三种类型：局部变量，成员变量和静态变量。

```java
public class Variable {
	int data = 88;                     // 成员变量
	static int num = 99;               // 静态变量
	final int FINALDATA = 33;          // 常量，Java 要求常量名必须大写
	public static void main(String[] args) {
        int a = 10;                    // 局部变量
        System.out.println(a);
    }
}
```

## 2. 表达式和语句

## 3. 注释

Java的单行注释和多行注释和C++一样：
```java
public void method() {
    // age 用于表示年龄
    int age = 18;

	/*
	age 用于表示年纪
	name 用于表示姓名
	*/
	int age = 18;
	String name = "名字";
}
```

Java多了一个文档注释：
```java
/**  
* Comment Demo  
*/  
public class Main {  
	private int age;  
	  
	public static void main(String[] args) {  
		System.out.printf(age);  
	}  
}
```
在 Intellij IDEA 中，按下 `/**` 后敲下回车键就可以自动添加文档注释的格式，`*/` 是自动补全的。

在命令行执行命令： `javadoc Demo.java` ，会生成一系列文档html，之后执行命令： `open index.html` 可以通过默认的浏览器打开文档注释。

![](Pasted%20image%2020230711121736.png)


# Java 面向对象

```Java
class Person {
	private String name;  
	private int age;  
	  
	public Person(String name, int age) {  // 构造函数  
		this.name = name;  
		this.age = age;  
	}
	
	public Person(String name) {  
		this(name, 18);                    // 调用另一个构造方法Person(String, int) 
	}

	public String getName() {  
		return this.name;  
	}
	
	public static void main(String[] args) {
        Person person = new Person("人");  // 类的声明使用new关键字
									       // 在堆中创建一个类实例
        System.out.println(person.name);
        System.out.println(person.age);
    }
}}
```

一个类可以包含：字段（Filed），方法（Method），构造方法（Constructor）。三种权限修饰符：public，private，protected。




## 1. 面向对象语法

### 1.1 方法

* 标准类库方法（内置方法）：Java 提供了大量预先定义好的方法供我们调用，也称为标准类库方法，或者内置方法。比如说 String 类的 `length()`、`equals()`、`compare()` 方法。
* 用户自定义方法
* 实例方法：没有使用 static 关键字修饰，但在类中声明的方法被称为实例方法，在调用实例方法之前，**必须创建类的对象**。
* 静态方法：有 static 关键字修饰的方法就叫做静态方法。当我们调用静态方法的时候，就**不需要 new 出来类的对象**，就可以直接调用静态方法。
* 抽象方法：没有方法体的方法被称为抽象方法，它总是在抽象类中声明，使用关键字 `abstract` 。

```java
public class MethodExample {
    public static void main(String[] args) {
	    // 对于实例方法的使用，必须先创建类对象
        MethodExample methodExample = new MethodExample();
        System.out.println(methodExample.add(1, 2));

		// 对于静态方法，可以直接调用
		System.out.println(minus(1,2));
	}

	// 实例方法
    public int add(int a, int b) {
        return a + b;
    }

	// 静态方法
	public static int minus(int a, int b) {
        return a - b;
    }
}

// 抽象类
abstract class AbstractDemo {
    abstract void display();   // 抽象方法
}

public class MyAbstractDemo extends AbstractDemo {
    @Override
    void display() {
        System.out.println("重写抽象方法");
    }

    public static void main(String[] args) {
        MyAbstractDemo myAbstractDemo = new MyAbstractDemo();
        myAbstractDemo.display();
    }
}

```

* native 本地方法：Java Native Interface (JNI)标准允许 Java 代码和其他语言编写的代码进行交互，调用不同语言编写的代码。使用 Java 与本地已编译的代码交互，通常会丧失平台可移植性。所以缺点是程序不再跨平台，程序不再是绝对安全。

#### 构造方法

在 Java 中，当一个类被实例化的时候，就会调用构造方法。
- 默认构造方法：如果一个构造方法中没有任何参数，那么它就是一个默认构造方法。目的主要是为对象的字段提供默认值。
- 有参构造方法：有参数的构造方法被称为有参构造方法，参数可以有一个或多个，有参构造方法可以为不同的对象提供值。
- 拷贝构造方法：参数为另一个对象的构造方法，可以把该参数的字段直接复制到新的对象中。
- 重载构造方法：通过提供不同的参数列表来重载构造方法，编译器会通过参数的类型和数量来决定应该调用哪一个构造方法。

**初始化字段只是构造方法的一种工作，它还可以做更多，比如启动线程，调用其他方法等。**


其中对于**代码初始化块**，编译器把代码初始化块放到了构造方法中。对象在初始化的时候会先调用构造方法，构造方法在执行的时候会把代码初始化块放在构造方法中其他代码之前。
```java
public class Car {
	// 代码初始化
    {
        System.out.println("代码初始化块");
    }
    
	Car() {
        System.out.println("构造方法");
    }
	
	// 与上面代码等价
	Car() {
		{
	        System.out.println("代码初始化块");
	    }
        System.out.println("构造方法");
    }
	
    public static void main(String[] args) {
        new Car();
    }
}
```


### 1.2 访问权限修饰符

在 Java 中，提供了四种访问权限控制：
- 默认访问权限（包访问权限）：即缺省，在同一个包内可以访问，其他包不能访问。
- public
- private
- protected：如果一个类的方法或者变量被 protected 修饰，对于同一个包的类，这个类的方法或变量是可以被访问的。对于不同包的类，只有继承于该类的类才可以访问到该类的方法或者变量。

其中，**类只可以用默认访问权限和 public 修饰**，方法和变量可以用四种修饰。

#### 包 package

包主要用来对类和接口进行分类。当开发Java程序时，可能编写成百上千的类，因此很有必要对类和接口进行分类。

源文件声明规则：
- 一个源文件中只能有一个public类
- 一个源文件可以有多个非public类
- 源文件的名称应该和public类的类名保持一致。例如：源文件中public类的类名是Employee，那么源文件应该命名为Employee.java。
- **如果一个类定义在某个包中，那么package语句应该在源文件的首行**。
- 如果源文件包含import语句，那么应该放在package语句和类定义之间。如果没有package语句，那么import语句应该在源文件中最前面。
- import语句和package语句对源文件中定义的所有类都有效。在同一源文件中，不能给不同的类不同的包声明。
- 类有若干种访问级别，并且类也分不同的类型：抽象类和final类等。这些将在后面两天中涉及到。除了上面提到的几种类型，Java还有一些特殊的类，如：内部类、匿名类


### 1.3 抽象类

定义抽象类的时候需要用到关键字 `abstract`，放在 `class` 关键字前。抽象类是不能实例化，但子类可以通过 `extends` 关键字来继承抽象类。抽象类中既可以定义抽象方法，也可以定义普通方法，但是抽象方法一定要定义在抽象类中。抽象类派生的子类必须实现父类中定义的抽象方法。

```java
public abstract class AbstractPlayer {
    abstract void play();
    
    public void sleep() {
        System.out.println("运动员也要休息而不是挑战极限");
    }
}

public class BasketballPlayer extends AbstractPlayer {
    @Override
    void play() {
        System.out.println("我是张伯伦，篮球场上得过 100 分");
    }
}
```


### 1.4 接口
接口通过 `interface` 关键字来定义，它可以包含一些常量和方法，
```java
public interface Electronic {
    // 常量
    String LED = "LED";
	
    // 抽象方法
    int getElectricityUse();
	
    // 静态方法
    static boolean isEnergyEfficient(String electtronicType) {
        return electtronicType.equals(LED);
    }
	
    // 默认方法
    default void printDescription() {
        System.out.println("电子");
    }
}

public class Computer implements Electronic {
    public static void main(String[] args) {
        new Computer();
    }
	
    @Override
    public int getElectricityUse() {
        return 0;
    }
}
```
- 接口中定义的变量会在编译的时候自动加上 `public static final` 修饰符，因此接口可以用来作为常量类使用。
- 没有使用 `private`、`default` 或者 `static` 关键字修饰的方法是隐式抽象的，在编译的时候会自动加上 `public abstract` 修饰符。
- 接口中允许有 `static` 方法和 `default` 方法。其中 `static` 方法只能通过接口名来调用，比如说 `Electronic.isEnergyEfficient("LED")`。 `default` 方法为实现该接口而不覆盖该方法的类提供默认实现。
- 接口的实现使用关键字 `implements` 。

作用：
- 使某些实现类具有我们想要的功能，比如说，实现了 Cloneable 接口的类具有拷贝的功能，实现了 Comparable 或者 Comparator 的类具有比较功能。
- **Java 原则上只支持单一继承，但通过接口可以实现多重继承的目的**。
- **实现多态**。

### 1.5 内部类
将一个类定义在另外一个类里面或者一个方法里面，这样的类叫做内部类。
- 成员内部类
- 局部内部类
- 匿名内部类
- 静态内部类

```java
public class Demo {
	// 成员内部类
	class MemberDemo {
        public void print() {
            System.out.println("成员内部类");
        }
    }

	// 静态内部类
	static class StaticDemo {
        public StaticDemo (){
            System.out.println("静态内部类");
        }
    }
	public Demo print() {
		// 局部内部类
        class LocalDemo extends Demo{
            private int age = 18;
        }
        return new LocalDemo();
    }

    public static void main(String[] args) {
		// 匿名内部类
        Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName());
            }
        });
        t.start();
    }
}
```

原因：每个内部类都能独立地继承一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响。

### 1.6 关键字
- `this` ：作为引用变量，指向当前对象
	- 调用当前类的方法。
	- `this()` 可以调用当前类的构造方法；
	- `this` 可以作为参数在方法中传递；
	- `this` 可以作为参数在构造方法中传递；
	- `this` 可以作为方法的返回值，返回当前类的对象。

- `super` ：每当创建一个子类对象的时候，也会隐式的创建父类对象，由 super 关键字引用。
	- - 指向父类对象；
	- 调用父类的方法；
	- `super()` 可以调用父类的构造方法。


- `static` ：**方便在没有创建对象的情况下进行调用**，包括变量和方法。
	- 静态变量：静态变量只在类加载的时候获取一次内存空间，节省空间。
	- 静态方法：静态方法属于这个类而不是这个类的对象，调用静态方法的时候不需要创建这个类的对象，静态方法可以访问静态变量。**静态方法不能访问非静态变量和调用非静态方法**。
	- 静态代码块
	- 静态内部类

- `final` ：最终态
	- 被 final 修饰的变量无法重新赋值。
	- 被 final 修饰的方法不能被重写。
	- 被 final 修饰的类无法被继承。

- `instanceof` ：判断对象是否符合指定的类型
	- 用法：`(object) instanceof (type)` 


## 2. 封装继承多态

面向对象的三大特点：封装、继承和多态。

**封装**是指利用抽象将数据和基于数据的操作封装在一起，使其构成一个不可分割的独立实体。

子类**继承**父类，也就拥有了父类中 protected 和 public 修饰的方法和字段，同时，子类还可以扩展一些自己的方法和字段，也可以重写继承过来方法。但是私有属性（私有变量、私有方法）不能被继承，但是可以通过继承父类的方法来访问父类的私有属性。

**private和final不被继承**。



**多态**是指在面向对象编程中，同一个类的对象在不同情况下表现出不同的行为和状态。

```java
public class Shape {
    public void draw() {
        System.out.println("形状");
    }
}

public class Circle extends Shape{
    @Override
    public void draw() {
        System.out.println("圆形");
    }
}

public class Line extends Shape {
    @Override
    public void draw() {
        System.out.println("线");
    }
}

public class Test {
    public static void main(String[] args) {
        Shape shape1 = new Line();
        shape1.draw();                 // 线
        Shape shape2 = new Circle();
		shape2.draw();                 // 圆形
    }
}
```



## 3. 反射与注解

在开发中使用注解和反射，有时候使用它们能让你事半功倍，简化代码提高编码的效率。

注解是一种趋势，一定程度上说，**框架 = 注解+反射+设计模式**。

### 3.1 反射

反射机制就是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为Java语言的反射机制。

**静态语言与动态语言**
1. 动态语言：是指在运行时可以改变其自身结构的语言，例如新的函数，对象，甚至代码可以被引进，已有的函数可以被删除或者结构上的一些变化。简单说即是在运行时代码可以根据某些条件改变自身结构。动态语言主要有 C#，Object-C，JavaScript，PHP，Python 等。
2. 静态语言：是指运行时结构不可改变的语言，例如 Java，C，C++等。

Java 不是动态语言，但是它可以称为**准动态语言**，因为 Java 可以利用反射机制获得类似动态语言的特性，Java 的动态性让它在编程时更加灵活。

反射机制允许程序在执行期借助于 Reflection API 取得任何类的内部信息，并能直接操作任意对象的内部属性以及方法等。

类在被加载完之后，会在堆内存的方法区中生成一个 Class 类型的对象，一个类只有一个 Class 对象，这个对象包含了类的结构信息。我们可以通过这个对象看到类的结构。

#### Reflection API

- java.lang.Class：代表一个类
- java.lang.reflect.Method：代表类的方法 
- java.lang.reflect.Field：代表类的成员变量
- java.lang.reflect.Construction：代表类的构造器


**在编译时不能确定类型，可以使用反射来实现。**



### 3.2 注解

注解是JDK5.0引入的新技术。注解并不是所装饰代码的一部分，它对代码的运行效果没有直接影响，由编译器决定该执行哪些操作。

#### 内置注解
- @Override ：重写方法
- @Deprecated ：不推荐程序员使用，但是可以使用，或者存在更好的方法
- @SuppressWarnings(value="") ：抑制警告



#### 元注解
元注解（meta-annotation）：负责注解其他注解
- @Target ：用于描述注解的使用范围（被描述的注解可以用在什么地方）
- @Retention ：表示需要在什么级别保存该注释信息，用于描述注解的生命周期。SOURCE < CLASS < RUNTIME
- @Documented ：说明该注解将被包含在javadoc中
- @Inherited ：说明子类可以继承父类中的该注解


#### 自定义注解
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation {
    //注解的参数：参数类型+参数名();
    //default "" 默认为空
    String name() default "";
    int age() default 0;
    int id() default -1;//如果默认值为-1，代表不存在

    String[] schools() default {"清华"};
}
```


# Java 泛型

在 Java 中，泛型是一种强类型约束机制，可以在编译期间检查类型安全性，并且可以提高代码的复用性和可读性。

```java
public class Box<T> {
    private T value;

    public Box(T value) {
        this.value = value;
    }

    public T getValue() {
        return value;
    }

    public void setValue(T value) {
        this.value = value;
    }
}
```

# Java 常用包

* JDK 的核心类使用`java.lang`包，编译器会自动导入；
* JDK 的其它常用类定义在`java.util.*`，`java.math.*`，`java.text.*`，……；


## 1. `java.lang`

`java.lang` 包是自动导入的，包的成员本身就在作用域内。主要包括包装类、String 类、Math 类、Class 类、Object 类等。

### 1.1 包装类

Java 中的基本数据类型不是面向对象的，因此 Java 为每个基本类型都提供了包装类。

### 1.2 String

`java.lang.String`，使用 String 类来定义一个字符串，字符串是常量，它们的值在创建之后不能更改。StringBuffer和StringBuilder支持可变的字符串。

### 1.3 Math

`java.lang.Math`，对数字进行数学操作，常用的方法有：
- `Math.pow(double a, double b)` ：计算a的b次方
- `Math.sqrt(double n)` ：计算平方根
- `Math.abs(int n)` ：计算绝对值
- `Math.ceil(double n)` ：返回大于等于 n 的最小整数。
- `Math.floor(double n)` ：返回小于等于 n 的最大整数。
- `Math.rint(double n)` ：返回最接近 n 的整数。
- `Math.round(T arg)` ：返回最接近 arg 的整数。
- `Math.random()` ：返回 `[0, 1)` 之间的随机数，double。

Math类的方法都声明为静态方法，因此需要通过类名来调用。


### 1.4 Class

`java.lang.Class`，注意Class类与关键字 `class` 的区别，Class 类的实例表示正在运行的 Java 应用程序中的类或接口，常常应用在反射中。

获取 Class 实例有三种方法：
1. 利用对象调用 `getClass()` 方法获取该对象的 Class 实例。
2. 使用 Class 类的静态方法 `forName(String className)`，用类的名字获取一个 Class 实例。
3. 运用 `.class` 的方式来获取 Class 实例。对于基本数据类型的封装类，还可以采用 `.TYPE` 来获取相对应的基本数据类型的 Class 实例。

```java
public class ClassTest {  
	public static void main(String[] args) throws ClassNotFoundException {  
		String s = new String("abc");  
		  
		Class classObj;  
		classObj = s.getClass();  
		System.out.println(classObj.getName()); // java.lang.String  
		  
		classObj = Integer.class;  
		System.out.println(classObj.getName()); // java.lang.Integer  
		  
		classObj = Class.forName("java.lang.String");  
		System.out.println(classObj.getName()); // java.lang.String  
	}  
}
```

### 1.5 Object

Object 类是所有类的父类，所有对象（包括数组）都实现这个类的方法，所以在默认的情况下，我们定义的类扩展自 Object 类。可以使用Object类型的变量引用任何类型的对象。

Object默认方法有：
-  `getClass()` - 获取类的class对象。
-  `hashCode()` - 获取对象的hashCode值
- `equals()` - 比较对象是否相等，比较的是值和地址，子类可重写以自定义。
- `clone()` - 克隆方法。
- `toString()` - 如果没有重写，应用对象将打印的是地址值。
- `notify()` - 随机选择一个在该对象上调用wait方法的线程，解除其阻塞状态。该方法只能在同步方法或同步块内部调用。
- `notifyall()` - 解除所有那些在该对象上调用wait方法的线程的阻塞状态。该方法只能在同步方法或同步块内部调用。
- `wait()` - 导致线程进入等待状态，直到它被其他线程通过notify()或者notifyAll唤醒。该方法只能在同步方法中调用。
- `finalize()` - 对象回收时调用

### 1.6 Enum

枚举（enum），是 Java 1.5 时引入的关键字，它表示一种特殊类型的类，继承自 `java.lang.Enum` 。
```java
public enum PlayerType {
    TENNIS,
    FOOTBALL,
    BASKETBALL
}
```

利用Enum类编写单例模式。
```java
// 不使用Enum类
public class Singleton {  
    private volatile static Singleton singleton;
    private Singleton (){}
    public static Singleton getSingleton() {
	    if (singleton == null) {
	        synchronized (Singleton.class) {
		        if (singleton == null) {
		            singleton = new Singleton();
		        }
	        }  
	    }  
	    return singleton;  
    }  
}

// 使用Enum类
public enum EasySingleton{
    INSTANCE;
}
```


## 2. `java.util`

`java.util` 提供了包含集合容器、遗留的集合类、事件模型、日期和时间实施、国际化和各种实用工具类。

使用时需要导入。

### 2.1 Collections工具类




## 3. `java.io`

### 3.1 字节流

#### OutputStream

`java.io.OutputStream` 是**字节输出流**的**超类**（父类）

#### InputStream

`java.io.InputStream` 是**字节输入流**的**超类**（父类）

### 3.2 字符流

#### Reader

`java.io.Reader`是**字符输入流**的**超类**（父类）

#### Writer

`java.io.Writer` 是**字符输出流**类的**超类**（父类）


### 3. 序列流

Java 的序列流（ObjectInputStream 和 ObjectOutputStream）是一种可以将 Java 对象序列化和反序列化的流。
- 序列化是指将一个对象转换为一个字节序列（包含对象的数据、对象的类型和对象中存储的属性等信息），以便在网络上传输或保存到文件中，或者在程序之间传递。在 Java 中，序列化通过实现 `java.io.Serializable` 接口来实现，只有实现了Serializable 接口的对象才能被序列化。
- 反序列化是指将一个字节序列转换为一个对象，以便在程序中使用。


# Java 集合容器

包名：`java.util`，Java 集合框架可以分为两条大的支线：
- `Collection` ：主要由 List、Set、Queue 组成。
- `Map` ：代表键值对的集合，典型代表就是 HashMap。
![](Pasted%20image%2020230704133720.png)

### 1. Collection接口

#### 1.1 List

##### ArrayList

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final int DEFAULT_CAPACITY = 10; // 默认容量为 10
    transient Object[] elementData; // 存储元素的数组，数组类型为 Object
    private int size; // 列表的大小，即列表中元素的个数
}
```
ArrayList 可以称得上是集合框架方面最常用的类了，可以和 HashMap 一较高下。
- **基于数组实现**，实现了自动扩容。
- 常用方法：
	-  `add()` - 插入
	- `remove(Object)` - 删除
	- `set(int index, E element)` - 修改
	- `indexOf(Object o)` - 正向查找

##### LinkedList

```java
/**
 * 链表中的节点类。
 */
private static class Node<E> {
    E item; // 节点中存储的元素
    Node<E> next; // 指向下一个节点的指针
    Node<E> prev; // 指向上一个节点的指针

    /**
     * 构造一个新的节点。
     *
     * @param prev 前一个节点
     * @param element 节点中要存储的元素
     * @param next 后一个节点
     */
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element; // 存储元素
        this.next = next; // 设置下一个节点
        this.prev = prev; // 设置上一个节点
    }
}
```

- 基于**链表实现**，封装了一个私有的静态内部类。
- 常用方法：
	-  `add()` - 插入链表，可以是最后，也可以指定位置
	- `remove(Object)` - 删除第一个节点
	- `set(int index, E element)` - 修改
	- `indexOf(Object o)` - 正向查找某个元素所在的位置
	- `get(int)` - 查找某个位置上的元素


##### Vector




#### 1.2 Queue

##### ArrayDeque

栈和队列都推荐使用更高效的ArrayDeque（双端队列），线程不安全。
- 底层基于数组实现
- 常用方法：
	- `push()`
	- `pop()`
	- `peek()` - 获取但不删除队首元素
	- `element()` - 获取但不删除队首元素
	- `add()` - 向队尾插入元素
	- `offer()` - 向队尾插入元素`
	- `remove()` - 获取并删除队首元素
	- `poll()` - 获取并删除队首元素`

添加，删除，取值都有两套接口，它们功能相同，区别是对失败情况的处理不同。一套接口遇到失败就会抛出异常，另一套遇到失败会返回特殊值（`false`或`null`）。


##### PriorityQueue

PriorityQueue 是 Java 中的一个基于优先级堆的优先队列实现，它能够在 O(log n) 的时间复杂度内实现元素的插入和删除操作，并且能够自动维护队列中元素的优先级顺序。
- 使用堆来实现，堆是完全二叉树
- 常用方法
	- `add(E e)` - 插入元素
	- `offer(E e)` - 插入元素
	- `element()` - 获取但不删除队首元素
	- `peek()` - 获取但不删除队首元素
	- `remove()` - 获取并删除队首元素
	- `poll()` - 获取并删除队首元素





#### 1.3 Set

##### HashSet

##### LinkedHashSet

##### TreeSet




### 2. Map接口

##### HashMap

HashMap 是 Java 中常用的数据结构之一，用于存储键值对。在 HashMap 中，每个键都映射到一个唯一的值，可以通过键来快速访问对应的值。
- **通过一个数组和链表或红黑树的组合来实现的**，初始大小是 16
- 哈希冲突时，采用拉链法将他们放在同一个索引的链表上。
- 常用方法：
	- `put(Object key, Object value)` - 添加键值对，也可以直接修改键值对。
	- `remove(Object key)` - 删除一个键值对
	- `get(Object key)` - 查找一个键对应的值

hash 方法源码：
```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
HashMap 的 hash 方法就是将 key 对象的 hashCode 值进行处理，得到最终的哈希值，并通过一定的算法映射到实际的存储位置上。

HashMap 扩容是通过 `resize` 方法来实现的，JDK 8 中融入了红黑树，链表长度超过 8 的时候，会将链表转化为红黑树来提高查询效率。
- 当负载因子超过**0.75**时触发扩容。
- 在进行扩容操作时，HashMap 会先将数组的长度扩大一倍，然后将原来的元素重新散列到新的数组中。
- 在重新散列元素时，如果一个元素的散列位置发生了改变，那么它需要被移动到新的位置。如果新的位置上已经有元素了，那么这个元素就会被添加到链表的末尾，如果链表的长度超过了阈值（8个），那么它将会被转换成红黑树。
- 

**HashMap是线程不安全的，因此在多线程环境下需要使用ConcurrentHashMap来保证线程安全。**

##### LinkedHashMap

为了保证键值对的插入顺序，提出了 LinkedHashMap ，其继承了 HashMap。LinkedHashMap 内部追加了双向链表，来维护元素的插入顺序。

可以使用 LinkedHashMap 来实现 LRU 缓存



##### TreeMap

TreeMap 由红黑树实现，默认根据 key 的自然顺序排列。


**Map的选择需要考虑以下因素**：
- 是否需要按照键的自然顺序或者自定义顺序进行排序。如果需要按照键排序，则可以使用 TreeMap；如果不需要排序，则可以使用 HashMap 或 LinkedHashMap。
- 是否需要保持插入顺序。如果需要保持插入顺序，则可以使用 LinkedHashMap；如果不需要保持插入顺序，则可以使用 TreeMap 或 HashMap。
- 是否需要高效的查找。如果需要高效的查找，则可以使用 LinkedHashMap 或 HashMap，因为它们的查找操作的时间复杂度为 O(1)，而是 TreeMap 是 O(log n)。


**HashTable**

- 底层数组+链表实现，无论key还是value都不能为null，线程安全，实现线程安全的方式是在修改数据时锁住整个HashTable，效率低，ConcurrentHashMap做了相关优化
- 初始size为11，扩容：newsize = olesize*2+1
- 计算index的方法：index = (hash & 0x7FFFFFFF) % tab.length






# Java 网络编程

`java.net` 包，




# Java 并发编程

多线程创建的方式：
```java
// 继承Thread类，并重写run方法
public class MyThread extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(getName() + ":打了" + i + "个小兵");
        }
    }
}

// 实现Runnable接口，并重写run方法。
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            try {//sleep会发生异常要显示处理
                Thread.sleep(20);//暂停20毫秒
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(
	            Thread.currentThread().getName() + "打了:" + i + "个小兵"
	        );
        }
    }
}

// 实现Callable接口，重写call()方法
public class CallerTask implements Callable<String> {
    public String call() throws Exception {
        return "Hello,i am running!";
    }

    public static void main(String[] args) {
        //创建异步任务
        FutureTask<String> task=new FutureTask<String>(new CallerTask());
        //启动线程
        new Thread(task).start();
        try {
            //等待执行完成，并获取返回结果
            String result=task.get();
            System.out.println(result);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

### 1. Runnable和Callable

`java.lang.Runnable` 是一个接口，在它里面只声明了一个 run()方法：
```java
public interface Runnable {
    public abstract void run();
}
```

Callable 位于 `java.util.concurrent` 包下，它也是一个接口，在它里面也只声明了一个方法，只不过这个方法叫做 `call()`：
```java
public interface Callable<V> {
    V call() throws Exception;
}
```
 Callable 一般情况下是配合 ExecutorService 来使用的，在 ExecutorService 接口中声明了若干个 submit 方法的重载版本


## 2. 并发编程问题

### 2.1 线程安全

在单线程环境中正常运行的代码，在多线程环境中可能会出现意料之外的结果。

利用Java的锁机制如synchronize和lock，加锁可以保证在同一时刻只有一个线程在执行同步代码块，释放锁之前会将变量刷回至主存，这样也就保证了可见性。

### 2.2 活跃性

如果加锁使用不当也容易引入其他问题，比如：死锁，活锁，饥饿问题。

### 2.3 性能问题

多线程有创建线程和线程上下文切换的开销，因此多线程并发不一定比单线程串行执行快。

## 3. 并发集合容器

### ConcurrentHashMap

- 底层采用分段的数组+链表实现，线程安全
- 通过把整个Map分为N个Segment，可以提供相同的线程安全，但是效率提升N倍，默认提升16倍。(读操作不加锁，由于HashEntry的value变量是 volatile的，也能保证读取到最新的值。)
- Hashtable的synchronized是针对整张Hash表的，即每次锁住整张表让线程独占，ConcurrentHashMap允许多个修改操作并发进行，其关键在于使用了锁分离技术
- 有些方法需要跨段，比如size()和containsValue()，它们可能需要锁定整个表而而不仅仅是某个段，这需要按顺序锁定所有段，操作完毕后，又按顺序释放所有段的锁
- 扩容：段内扩容（段内元素超过该段对应Entry数组长度的75%触发扩容，不会对整个Map进行扩容），插入前检测需不需要扩容，有效避免无效扩容。


## 4. 线程池



# 运行原理

## 1. 运行环境
![](Pasted%20image%2020230711115623.png)
只需要运行java程序，安装JRE（Java运行时环境，Java Runtime Environment）即可。需要编写Java代码，安装JDK。

JVM 是一种逐行运行 Java 程序的软件。


## 2. 编译过程
![](Pasted%20image%2020230711115408.png)

1. 编译字节码：`javac Sample.java`
2. 反编译字节码：`javap -c Sample.class`
3. 执行：`java Sample`




ConcurrentHashMap

垃圾回收




## 3. JVM

### GC

Java 虚拟机（JVM）提供了一种自动回收内存的机制，它一般会在内存空闲或者内存占用过高的时候对那些没有任何引用的对象不定时地进行回收。
![](Pasted%20image%2020230704133707.png)


判定一个对象是否是“垃圾”，即判定一个对象的存活与否，算法有两种：**引用计数法**和**根搜索算法**。
- 引用计数算法：一个对象被创建之后，系统会给这个对象初始化一个引用计数器，当这个对象被引用了，则计数器 +1，而当该引用失效后，计数器便 -1，直到计数器为 0，意味着该对象不再被使用了，则可以将其进行回收了。但是无法处理循环引用，现在基本不再使用了。
- 根搜索算法：根搜索算法的中心思想，就是从某一些指定的根对象（GC Roots）出发，一步步遍历找到和这个根对象具有引用关系的对象，然后再从这些对象开始继续寻找，从而形成一个个的引用链（其实就和图论的思想一致），然后不在这些引用链上面的对象便被标识为引用不可达对象，也就是我们说的“垃圾”，这些对象便需要回收掉。

回收算法有如下几种：

- 标记-清除（Mark-Sweep）算法
- 标记-整理（Mark-Compact）算法
- 复制（Copying）算法
- 分代收集算法



# 工具

## JDK

[Java Downloads | Oracle 中国](https://www.oracle.com/cn/java/technologies/downloads/)

选择长期支持版本JDK17就好。

## IDEA

Java的集成开发工具。
![](Pasted%20image%2020230706120826.png)






  

# 参考

- [To Be Better Javaer](https://tobebetterjavaer.com/home.html)