## JDK、JRE、JVM
- JDK：Java Development Kit
- JRE：Java Runtime Environment
- JVM：JAVA Virtual Machine
![](https://raysuner-image-bed.oss-cn-hangzhou.aliyuncs.com/202406290004189.png)
## 编写第一个Java程序
HelloWorld.java
```java
public class HelloWorld{
  public static void main(String[] args) {
    System.out.print("hello world");
  }
}
```
可能遇到的问题：
1. 类名需要和文件名重合
2. 注意大小写
3. 语句后需要加分号
## Java数据类型
所有的字符本质上还是数字，Unicode编码，占用2个字节，65536 = 2 ^ 16
```java
char c = '\u0061'
```
上面`\u0061`表示的是小写字母a，`\u`开头表示是unicode编码，其中61是以16进制的方式来表达的，转换成十进制刚好是97，对照unicode表则刚好是
## 类型转换
注意点：
- 高转低时需要用到强制类型转换，`byte, short, char -> int -> long -> float -> double`由占据字节大的类型向占据字节小的类型转换时，需要使用强制类型转换，反过来无需使用强制转换，会发生隐式类型转换。
- 不能把对象类型转为不相干的类型
- 注意转换过程可能存在内存溢出和精度问题
- 不能对布尔类型进行转换
- 数字较大时注意溢出问题
## 