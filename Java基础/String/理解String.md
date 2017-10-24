# 理解String

标签（空格分隔）： Java String

---

+ String 是不可变的: String 类是被final 修饰的，也就是说，String 类是不可被继承的；String 的值也是不可改变的。
 ```
 public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];  
    // 存储String 的值是一个char 数组，为了让String的值不可变，String 类中很小心的处理这个数组，在整个String类中都没有改变数组中值的方法，且为了防止使用者的勿用，让String使用final修饰，让其不可继承。
    
    ......
    
    public native String intern();
    // intern() 方法是让String 对象的字符序列转存到 字符串常量池中，并返回在字符串常量池中的引用。（让JVM扩充常量池：若字符序列在常量池中不存在，则扩充常量池，将字符串序列存入常量池）
    }
    
 ```
+  使用 String 不一定创建对象：如
 ```
String a = "aaaa";  
String b = a + "bbbb"; // 因为a是变量，JVM对b的优化是将“+”转化为StringBuilder.apped()操作，返回b的是一个新的String对象（JDK1.5前是StringBuffer.append）
String c = "aaaabbbb"; 
//b == c false
 ```

+  使用 new String() 时一定创建对象：用构造器创建的String，返回给变量的是“指向常量池字符串的引用的引用”（即变量指向一个在堆中指向常量池中字符串的引用） 。

+ StringBuilder 和 StringBuffer:
    1. StringBuffer是线程安全的，StringBuilder是不安全的，从StringBuffer方法前的synchronized 可以看出。
    2. StringBuilder 和 StringBuffer 是可变的，这2者存放字符的 char[] 是可变的，这样就不会像String一样在做字符拼接时不断的新建String对象，这可以提升字符拼接的效率。
    3. JVM 都预处理的 "+", 做成StringBuilder.append(),那为什么还要手动使用StringBuilder呢？
    ```
    String s1 = "aaaaa";  
    String s2 = "bbbbb";  
    String r = null;  
    int i = 3694;  
    r = s1 + i + s2;   // JVM 在此处会new StringBuilder 处理字符串拼接
              
    for(int j=0;i<10;j++){  
        r+="23124";  // JVM 会在每次循环中new StringBuilder 处理字符串拼接，所以为了不每次新建StringBuilder对象，还需要手动使用StringBuilder
    }  
    ```

+ 通过代码反编译查看编译器对字符进行"+"号时的优化处理：
```
//原代码
public class Test {

    public static void main(String args[]) {
        String str = "abc";
        str = str + "haha" + 1;
        for(int i = 0; i < 20; i++){
            str += i;
        }
    }
}
```
![Test反编译][1]
 


  [1]: https://github.com/FarmerShao/treasure/blob/master/Java%E5%9F%BA%E7%A1%80/String/Test%E5%8F%8D%E7%BC%96%E8%AF%91.png