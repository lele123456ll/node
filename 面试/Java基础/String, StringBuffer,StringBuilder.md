# String-StringBuffer-StringBuilder区别

## 可变性

```c++
String底层使用了final关键字-- private final char value[], 不可变
StringBuilder与StringBuffer都继承⾃AbstractStringBuilder类, 底层为char value[] 可变
```

## 线程安全性

```c++
String是线程安全的
StringBuffer添加了同步锁也是线程安全的
StringBuilder是线程不安全的
```

## 性能

```
每次对 String 类型进⾏改变的时候，都会⽣成⼀个新的 String 对象，然后将指针指向新的 String
对象。
StringBuffer 每次都会对 StringBuffer 对象本身进⾏操作，⽽不是⽣成新的对象并改变对象
引⽤。
相同情况下使⽤ StringBuilder 相⽐使⽤ StringBuffer 仅能获得 10%~15% 左右的性能提
升，但却要冒多线程不安全的⻛险。
```

## 总结

```
1. 操作少量数据用String
2. 单线程操作大量数据用StringBuilder
3. 多线程
```







