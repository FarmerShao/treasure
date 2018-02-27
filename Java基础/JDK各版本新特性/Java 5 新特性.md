# Java 5 新特性

标签（空格分隔）： 新特性 java
---
## [新特性总览][1]
 [1. 泛型（Generics）](#1)
 [2. 增强for循环（Enhanced for Loop）](#2)
 [3. 自动拆装箱（Autoboxing/Unboxing）](#3)
 [4. 枚举（Typesafe Enums）](#4)
 [5. 可变长参数（Varargs）](#5)
 [6. 静态导入（Static Import）](#6)
 [7. 注解/元注解（Annotations/MetaData）](#7)

### **<span id="1">泛型（Generics）</span>**
泛型为Collection提供了编译时的类型安全检测，并消除了类型强制转换的工作。
#### **泛型使用规则：**
+ 所有泛型方法声明都有一个类型参数声明部分（由尖括号分隔），该类型参数声明部分在方法返回类型之前（在下面例子中的<E>）。多个类型参数声明使用逗号“,”分隔
```
//需要使用<E> 在返回类型（此处为 void） 前声明泛型的类型参数
public static <E> void printArray( E[] inputArray ){
    // 输出数组元素
    for ( E element : inputArray ){
        System.out.printf( "%s ", element );
    }
    System.out.println();
}
```
+ 泛型可以作为返回类型，参数使用。
+ 泛型通配符"?"：当不关心泛型的具体类型时可以使用该通配符。
+ 类型的局部限制 <? extends Number>：指定某一类型及其子类。例子中Number都可以转换为其它的具体的引用类型。
+ 泛型的使用注意事项：
    + 泛型不能作为Java 的基本数据类型使用，即使用泛型声明的泛型方法的参数不能使用基本数据类型的变量传入。
    
    ```
    public static <E> void printArray( E[] inputArray ) {
        .....
    }
    public static void main( String args[] ) {
        int[] intArray = { 1, 2, 3, 4, 5 };
        //注意这里是错误的，因为数组是 int 类型的，而printArray 方法的参数是泛型
        printArray( intArray  ); // 传递一个整型数组
    }
    ```
    + 在编译时实会进行`类型擦除`操作的，即Java源码中的范型信息只允许停留在编译前期，而编译后的字节码文件中将不再保留任何的范型信息。也就是说，范型信息在编译时将会被全部删除，其中范型类型的类型参数则会被替换为Object类型，并在实际使用时强制转换为指定的目标数据类型。所以`不能对具体的泛型类型进行instatnceof 操作`，即 (? instatnceof FX<Number>) 是不正确的。
    + 不能对一个确切的泛型创建数组，但是可以创建使用通配符声明的泛型数组。
    
    ```
    //这里是错误的
    List<String>[] lsa = new List<String>[10]; 
    //这里是正确的
    List<？>[] lsa = new List<？>[10];
    ```


----------


### **<span id="2">增强for循环（Enhanced for Loop）</span>**
#### **for-each 优缺点：**
优点：
1. 代码更加的优美，结构更加的清晰。
2. 避免了在使用使用iterator迭代时，next()方法的误用。
缺点：
1. 无法用于数组/集合的筛选过滤，即不能再遍历的过程中执行remove操作。
2. 无法获取元素的index

```
for ( String str : inputArray ){
    System.out.printf( "%s ", str );
}
```


----------


### **<span id="3">自动拆装箱（Autoboxing/Unboxing）</span>**
装箱： 将基本类型数据封装为一个封装类对象：例 int --> Integer
拆箱： 将一个基本类型的封装类转换为一个基本类型的数据：例 Integer --> int
自动拆装箱： 在集合Collection 中只能存放引用类对象，而在表达式计算时只能使用基本类型数据。在没有加入自动拆装箱特性之前，将一个基本类型数据存入集合中需要将基本类型的封装成一个封装类对象；在取出集合中的基本类型封装类做计算时，将封装类对象拆箱为一个基本数据类型。加入自动拆装箱特性之后，编码时就无需再执行上述重复的操作，Java程序会自动执行上述操作，也就是说，大部分时候封装类和基本类型基本都是一样的，封装类也就是多了一些Object所有的一些特性，在基本功能上（数字或符号本身的职责上）都一致。 


----------


### **<span id="4">枚举（Typesafe Enums）</span>**
枚举是一种新的数据类型，它既有类(class)的特性，又比类多了一些约束。
#### **枚举的定义：**
在引入枚举之前，Java 定义一组常量都是以下这种格式：
```
/*定义一个类，然后在类里面定义各个常量
*不足之处： 不能保证各个常量值的唯一性，即在下面这个例子中，MONDAY,TUESDAY等常量的int 值是可以一样的。
*/
public class DayDemo {
    public static final int MONDAY = 1;
    public static final int TUESDAY = 2;
    public static final int WEDNESDAY = 3;
    public static final int THURSDAY = 4;
    public static final int FRIDAY = 5;
    public static final int SATURDAY = 6;
    public static final int SUNDAY = 0;
}
```
在引入枚举后，可以将上面的常量形式写成下面的枚举：
```
/*
* 使用枚举，代码变得更加简洁。由于Java 语义上的支持，使得枚举比常量更加的安全---
* 每一个枚举在程序中都是单例的，不可通过构造器创建新的实例，即使是使用反射也无法新建枚举实例。
* 枚举的使用也是非常的便捷，例：Day.MONDAY 
*/
enum Day{
    MONDAY,TUESDAY,WEDNESDAY,THURSDAY,FRIDAY,SATURDAY,SUNDAY
}
```
#### **枚举与类的比较**
##### **不同点：**
+ 首先枚举与类在Java中使用的关键字不同， 前者是 enum，后者是 class。
+ 枚举不能继承其它类，这是因为枚举已经继承了Enum<T> 这个抽象类。
+ 枚举的构造函数只能是 privete 的
+ 枚举不能创建新的实例，即使使用反射技术也不能创建，都是由JVM自动创建，且是单例的。
+ 枚举类是final 类型的，不能再被继承
+ 枚举默认实现了 Comparable<E> 接口，是通过比较枚举的 ordinal 这个值。
##### **相同点：**
+ 可以实现接口
+ 可以有属性
+ 可以添家自定义方法
#### **枚举的抽象类 Enum&lt;T&gt;**
Enum 的属性：
|属性|说明|
|:---:|:---:|
|name|枚举的名称，即枚举声明时的字符串；入上述的 DAY.MONDAY 枚举，则其name="MONDAY" |
|ordinal|枚举声明时的 index 值，从0起始，如上述的Day.MONDAY 的ordinal =0；改值一般不怎么使用，但是在EnumSet和EnumMap中会使用到这个属性；
|
Enum 的方法：
|返回类型|方法名|说明|
|:---:|:---:|:---:|
|String|name()|返回枚举的name属性值|
|String|toString()|返回枚举的name属性值|
|int|ordinal()|返回枚举的ordinal属性值|
|int|compareTo()|枚举实现了Comparable接口，判断的依据是 ordinal 的值，返回值为 self.ordinal - other.ordinal|
|T|static valueOf(Class&lt;T&gt; enumType,String name)|通过枚举类型和枚举的name，获取对应的枚举实例；编译器为枚举实现的valueOf(String name)方法内部就是调用的这个方法|
|Class&lt;E>|getDeclaringClass()|获取当前的枚举的class|
|void|readObject()|直接抛出InvalidObjectException异常，这是为了阻止默认的反序列化，保证枚举实例的单例性|
|void|readObjectNoData()|直接抛出InvalidObjectException异常，这是为了阻止默认的反序列化，保证枚举实例的单例性|
此处通过代码说明下 ordinal:
```
public class Demo{
    enum Day {
        MONDAY,TUESDAY,WEDNESDAY,THURSDAY,FRIDAY,SATURDAY,SUNDAY
    }

    public static void main(String[] args) {
        for (Day day : Day.values()){
            System.out.print(day.name() + ":" + day.ordinal() + "  ");
        }
    }
    
    /**
    * 控制台打印结果：
    * MONDAY:0  TUESDAY:1  WEDNESDAY:2  THURSDAY:3  FRIDAY:4  SATURDAY:5  SUNDAY:6 
    */
}
```
##### **枚举与单例模式：**
饿汉式：
```
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    private Singleton() {
    }
    public static Singleton getInstance(){
        return INSTANCE;
    }
}
```
双重检测模式：
```
public class Singleton {
    private static volatile Singleton instance = null;
    private Singleton() {
    }
    public static Singleton getInstance(){
        if(instance == null) {
            synchronized (Singleton.class){
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
静态内部类模式：
```
public class Singleton {
    private static class SingletonInner{
        public static  Singleton INSTANCE = new Singleton();
    }
    private Singleton() {
    }
    public static Singleton getInstance(){
        return SingletonInner.INSTANCE;
    }
}
```
上述三种单例模式都能够在多线程环境下成功创建单例，但是“饿汉式”的单例不是延迟加载，若是需要创建的单例是一个需要很多资源的类，那么会在程序启动时因创建这个单例而消耗很多时间，使得程序的启动时间过长。而“双重检测模式”和“静态内部类模式”即具备了线程安全特性，还提高了效率。
但是上述三种模式都不能保证序列化安全和反射安全：
1. 序列化安全：序列化可能会破坏单例模式，比较每次反序列化一个序列化的对象实例时都会创建一个新的实例，解决方案如下：
```
public class Singleton implements Serializable {     
    public static Singleton INSTANCE = new Singleton();     
    protected Singleton() { }  
    //反序列时直接返回当前INSTANCE，其它几种模式的解决方案类似
    private Object readResolve() {     
         return INSTANCE;     
    }    
}   
```
2. 使用反射强行调用私有构造器，解决方式可以修改构造器，让它在创建第二个实例的时候抛异常
```
public static Singleton INSTANCE = new Singleton();     
private static volatile  boolean  flag = true;
private Singleton(){
    if (flag) {
        flag = false;   
    } else {
        throw new RuntimeException("Hey boy, the INSTANCE is already exists. Don't recreate it.");
    }
}
```
为了保证单例的效率、线程安全、反射安全以及序列化安全，我们写了很多的代码。现在有一种方式很简单、很优雅的就为我们提供了上述的特性，那就是通过枚举来实现一个单例。
```
public enum Singleton{
    INSTATNCE
}
```
枚举是如何实现反射安全和序列化安全的：
1. 序列化安全： Jvm在给枚举序列化时只是将枚举的name作为输出；在反序列化时，JVM会通过valueOf(String name)方法，来获取对应的枚举对象。
2. 反射安全： 
```
//这是JDK的Constructor 类的newInstance()方法的源码
public T newInstance(Object ... initargs)
    throws InstantiationException, IllegalAccessException,
           IllegalArgumentException, InvocationTargetException
{
    if (!override) {
        if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
            Class<?> caller = Reflection.getCallerClass();
            checkAccess(caller, clazz, null, modifiers);
        }
    }
    //实例化时，判断是否为Enum 若是枚举则抛出 IllegalArgumentException，
    //这也就阻止了通过反射调用私有构造器来创建枚举实例
    if ((clazz.getModifiers() & Modifier.ENUM) != 0)
        throw new IllegalArgumentException("Cannot reflectively create enum objects");
    ConstructorAccessor ca = constructorAccessor;   // read volatile
    if (ca == null) {
        ca = acquireConstructorAccessor();
    }
    @SuppressWarnings("unchecked")
    T inst = (T) ca.newInstance(initargs);
    return inst;
}
```


----------

### **<span id="5">可变长参数（Varargs）</span>**
当方法的参数需要接收不定长的参数时，我们可以使用数组的形式传递参数：
```
public static void main(String[] args) {
    ......
}
```
在JDK 5 中引入了更加方便的特性来支撑方法接收不定长的参数---可变长参数，使用方式入下：
```
public static <T extends Number> void testMethod(T... args){
    System.out.println(args.length);
}
```
可变长参数可以在一定程度上可以看做是一个数组，但是在方法调用时无需像数组参数那样需要创建一个数组，只需将要传入的参数以普通的参数那样通过","分隔即可：
```
testMethod(1,2,3);
```
**注意事项：**
+ 可变长参数必须在方法的参数列表最后声明。例：void testMethod(T... args，Integer i){} 这样声明时错误的
+ 在方法的重载中不能存在只有可变长参数和数组参数差异的方法，如：
```
public static <T extends Number> void testMethod(T... args){
    System.out.println(args.length);
}
//编译报错，因为编译器会将可变长参数最终转化为一个数组参数，这就违背了Java 方法重载的规范。
public static <T extends Number> void testMethod(T[] args){ 
    System.out.println(args.length);
}
```
+ 因为基本类型的自动拆装箱特性的存在，切勿同时重载基本数据类型和封装类的方法存在，否则在调用方法时编译器会因课同时call 2个方法而编译报错。
```
public static void main(String[] args) {
    //报错，编译器不知道该调用那个方法，即使是参数通过Integer 封装类的形式传入。
    testMethod(new Integer(1),new Integer(2),new Integer(3));
}

//这样重载方法不违背Java 的重载规范
public static void testMethod(Integer... args){
    System.out.println(args.length);
}

public static void testMethod(int... args){
    System.out.println(args.length);
}
```


----------
### **<span id="6">静态导入（Static Import）</span>**
当Java 未引入"静态导入"特性时，调用静态成员时必须指明成员所属的类：
```
double r = Math.cos(Math.PI * theta);
```
引入"静态导入"后，只需在类的import部分导入静态成员，后续使用时就可以不再指明成员所属的类：
```
import static java.lang.Math.PI;
//或者是导入Math下的所有静态成员
import static java.lang.Math.*;
//调用时无需再指明所属类，直接使用静态成员即可
double r = cos(PI * theta);
```

----------
### **<span id="7">注解/元注解（Annotations/MetaData）</span>**
#### **注解声明：**
```
//注解的声明和接口的声明基本一致，只需在interface关键字前添加一个'@'符号即可
public @interface TestDemo {
    /*
    *   注解中声明的每一个方法都是注解的一个“元素”，若只有一个元素时，最好将该元素声明为value;
    *   1.方法的返回值只能是:String，基本类型，Class，enum，annotation和前面几种类型的数组(这些数据
    *   类型都能在编译期明确值)
    *   2.每个元素都可以设置默认值，通过关键字 default 定义；数组的默认值通过{}括起来。
    *   所有的值都不能是null
    */
    String value() default "test" ;  
    Day[] days() default {Day.MONDAY, Day.TUESDAY};
    Target testDemo() default @Target(ElementType.ANNOTATION_TYPE);
}
```
#### **Annotation(Metadata)：**
所有Annotation(Metadata)都是只能作用在注解上的。
***@Target***：指明当前注解的可使用范围

|返回值类型|元素名|说明|
|:---:|:---:|:---:|
|ElementType[]|value|指定可使用范围|

```
//ElementType JDK源码
public enum ElementType {
    /** 
        Class, interface (including annotation type), or enum declaration 
        类、接口（包括注解类型）或enum声明
    */
    TYPE,
    /** 
        Field declaration (includes enum constants) 
        字段(域)声明，包括enum实例
    */
    FIELD,
    /** 
        Method declaration 
        方法声明
    */
    METHOD,
    /** 
        Formal parameter declaration 
        参数声明
    */
    PARAMETER,
    /** 
        Constructor declaration 
        构造方法声明
    */
    CONSTRUCTOR,
    /** 
        Local variable declaration 
        本地变量声明
    */
    LOCAL_VARIABLE,
    /** 
        Annotation type declaration 
        注解声明
    */
    ANNOTATION_TYPE,
    /** 
        Package declaration 
        包声明
    */
    PACKAGE,
    /**
     * Type parameter declaration
     * 类型参数声明
     * @since 1.8
     */
    TYPE_PARAMETER,
    /**
     * Use of a type
     * 类型使用声明
     * @since 1.8
     */
    TYPE_USE
}
```
***@Retention***：用来约束注解的生命周期
|返回值类型|元素名|说明|
|:---:|:---:|:---:|
|RetentionPolicy|value|指明注解的生命周期|
```
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     * 注解会被编译器给丢弃，只能存在源码中
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     * 注解会被编译器记录到class文件中，但是会被VM运行时所丢弃。注解的默认行为就是这个级别
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *  注解会被编译器记录到class文件中，同事VM运行时也会保留注解信息，所以注解可以被反射获取到。
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```
还有很多MetaData的元注解，比如@Documented等只能作用在注解上的注解，就不一一解释了。注解其实大多数情况下都是作为一种标记来使用的。


  [1]: https://docs.oracle.com/javase/1.5.0/docs/guide/language/index.html
