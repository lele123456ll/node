# Java基础

## 面向对象特性

- 封装 : `封装把一个对象属性私有化, 提供一些公有访问给外界访问`

    - **把该隐藏的隐藏起来，该暴露的暴露出来，这就是封装性的思想**

- 继承 : `是使用已存在的类作为基础建立新的类, extends。新的类拥有父类的所有属性与方法`

- 多态

    ```
    所谓多态就是指程序中定义的引⽤变量所指向的具体类型和通过该引⽤变量发出的⽅法调⽤在编
    程时并不确定，⽽是在程序运⾏期间才确定。
    即⼀个引⽤变量到底会指向哪个类的实例对象，该引⽤变量发出的⽅法调⽤到底是哪个类中实现的⽅法，必须在由程序运⾏期间才能决定。
    
    在 Java 中有两种形式可以实现多态：继承（多个⼦类对同⼀⽅法的重写）和接⼝（实现接⼝并
    覆盖接⼝中同⼀⽅法）。
    ```

## [泛型与类型擦除](https://www.yuque.com/docs/share/13008461-c3e8-4896-9e72-653f0adcff97?# 《泛型的类型擦除》)



## 反射原理, 优缺点

### 优点

1. **增加程序的灵活性，避免将程序写死到代码里。**
2. **代码简洁，提高代码的复用率，外部调用方便**
3. **对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法**

### 缺点

1. **性能问题**
2. **使用反射会模糊程序内部逻辑**
3. **安全限制**
4. **内部暴露**

## Static与final关键字

### [**final**](https://www.yuque.com/docs/share/4bb0b6fd-7dff-4bbc-9c6b-1f09220a2465?# 《final》)

### [static](https://www.yuque.com/docs/share/2f9563fb-ae1d-4e50-9b98-7198565b1760?# 《static使用（从生命周期的角度理解！！！）》)

## [String, StringBuffer,StringBuilder](https://www.yuque.com/docs/share/45075414-bf31-45a1-98a3-9ae8dec8ec1d?# 《String, StringBuffer,StringBuilder》)

## BIO, NIO, AIO

- [bio, nio, aio](https://developer.aliyun.com/article/726698)

## Object类方法

```
registerNatives
getClass
hashCode
equals
clone
toString
notify
notifyAll
wait(long)
wait(long, int)
wait()
finalize
```



## 自动拆线与装箱

```java
装箱就是自动将基本数据类型转换为包装器类型；
拆箱就是自动将包装器类型转换为基本数据类型。

Integer i = 10;  //装箱
int n = i;   //拆箱
```

| int（4字节）    | Integer   |
| :-------------- | --------- |
| byte（1字节）   | Byte      |
| short（2字节）  | Short     |
| long（8字节）   | Long      |
| float（4字节）  | Float     |
| double（8字节） | Double    |
| char（2字节）   | Character |
| boolean（未定） | Boolean   |

## ValueOF&xxxValue

```java
static final int low = -128;
static final int high;
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}

```



