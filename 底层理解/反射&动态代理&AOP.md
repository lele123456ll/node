# 反射&动态代理&AOP

## 反射

> Java反射就是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；并且能改变它的属性

- 反射之前

    ```java
      //1.创建对象
      Person person = new Person("lle", 14);
      //2.通过对象调用其内部的属性,方法
      person.show();
      person.age=10;
      System.out.println(person);
      //在person类的外部,不可以通过Person类的对象调用其内部的私有结构
      //比如name属性,showNation()方法以及私有的构造器
    ```

- 反射之后

    ```java
    Class clazz = Person.class
        
    //1.通过反射创建对象
    Constructor cons = clazz.getConstructor(String.class, int.class);
    Object o = cons.newInstance("lele", 12);
    Person p = (Person)o;
    System.out.println(p.toString());
    
    //2.通过反射,调用属性方法
    //调用属性
    Field age = clazz.getDeclaredField("age");
    age.set(p,10);
    System.out.println(p.toString());
    //调用方法
    Method show = clazz.getDeclaredMethod("show");
    show.invoke(p);
    System.out.println("************************************");
    //通过反射,可以调用其内部的私有结构
    //比如name属性,showNation()方法以及私有的构造器
    
    //调用私有构造器
    Constructor cons1 = clazz.getDeclaredConstructor(String.class);
    cons1.setAccessible(true);
    Person p1 = (Person)cons1.newInstance("kk");
    System.out.println(p1.toString());
    
    //私有属性
    Field name = clazz.getDeclaredField("name");
    name.setAccessible(true);
    name.set(p1,"henry");
    System.out.println(p1.toString());
    
    //私有方法
    Method showNation = clazz.getDeclaredMethod("showNation");
    showNation.setAccessible(true);
    String invoke = (String) showNation.invoke(p1);
    System.out.println(invoke);
    ```

- Class类

    **获取Class类对象**

    1. `调用运行类的属性 .class  -->Class clazz = 类名.class`
    2. `通过运行类的对象,调用getClass()`
    3. `调用Class的静态方法:forName(String classPath)`

    **Class类方法**

    ```java
    获取类属性、方法、构造器..
    
    //属性:
    1. getFields()方法获取当前运行类和父类中权限为public的属性
    2. getDeclaredFields()获取当前类的所有属性
    //属性的权限修饰符 数据类型 变量名:
    f.getModifiers() 
    f.getType()  
    f.getName()  
        
        
    //方法
    1. getMethods() 获取当前类和其父类的所有权限修饰符为public的方法
    2. getDeclaredMethods() 获取当前类的所有方法
    Method eat = userClass.getMethod("eat", String.class);
    //获取方法中的注解 权限修饰符 返回值类型 方法名 形参列表 异常:
        getAnnotations() 
        getModifiers()
        getReturnType()
        getName()
        getParameterTypes()
        getExceptionTypes()
            
           
    //构造器
    1. getConstructors()   获取当前类中权限修饰符为public的构造器
    2. getDeclaredConstructors()   获取当前类中的所有构造器
    ```

- ClassLoader

    ```java
    1. 系统类加载器
    2. 扩展类加载器
    3. 引导类加载器
    
    class A{
    }
    //对于自定义类，使用系统类加载器进行加载
    ClassLoader classLoader = A.class.getClassLoader();
    //调用系统类加载器的getParent():获取扩展类加载器
    ClassLoader parent = classLoader.getParent();
    //调用扩展类加载器的getParent():无法获取引导类加载器
    //引导类加载器主要负责加载java的核心类库，无法加载自定义类的。
    ClassLoader parent1 = parent.getParent();
    ClassLoader classLoader3 = String.class.getClassLoader();    
    ```

## 动态代理

> 代理模式是一种设计模式，能够使得在不修改源目标的前提下，额外扩展源目标的功能。即通过访问源目标的代理类，再由代理类去访问源目标。这样一来，要扩展功能，就无需修改源目标的代码了。只需要在代理类上增加就可以了。

![动态代理](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/501/%E4%BB%A3%E7%90%86.jpg)

**实例**

```java
// 接口
public interface human{
	String getBelief();
	void eat(String food);
}
// 实现类 == 被代理类
class SuperMan implements Human{
    @Override
    public String getBelief() {
    	return "i can fly";
    }
    @Override
     public void eat(String food) {
         System.out.println("i like" + food);
     }
}

//问题
1. 如何根据被加载到内存的得代理类, 动态的创建代理类的对象  ==> 反射
2. 通过代理类对象调用方法A时, 如何动态的去调用被代理类中的方法  ==> 反射

    
//代理类工厂
Class ProxyFactory {
    //通过被代理类的对象获取代理类的对象
    public static Object getInstance(Object obj){  
        MyHandler handler = new MyHandler();
        //绑定被代理类对象
        handler.bind(obj);
        return  Proxy.newProxyInstance(obj.getClass().getClassLoader(),obj.getClass().getInterfaces(),handler);
    }
}
class MyHandler implements InvocationHandler{

    private Object obj; // 被代理类的对象
    public void bind(Object obj){
        this.obj = obj;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //method : 为代理类对象调用的方法,此方法也就作为了被代理类对象要调用的方法
        //obj : 被代理类的对象
        Object returnValue = method.invoke(obj, args);
        return returnValue;
    }
}
```

## AOP(动态代理实现)

> (1)面向切面编程(方面)，利用AOP可以对业务逻辑的各个部分进行隔离，从而使得
> 业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
> (2)通俗描述:**不通过修改源代码方式，在主干功能里面添加新功能**

- AOP术语

    ```
    1. 连接点
     类里面哪些方法可以被增强,这些方法称为连接点
    2. 切入点
     实际被增强的方法,称为切入点
    3. 通知(增强)
     (1)实际增强的逻辑部分称为通知(增强)
     (2)通知有多种类型
     ··前置通知
     ··后置通知
     ··环绕通知
     ··异常通知
     ··最终通知
    4. 切面
     是动作
     (1)把通知应用到切入点的过程
    ```

- AOP操作

    ```
    1. Spring 框架一般都是基于AspectJ实现AOP操作。
        什么是Aspect?
        Aspect不是Spring组成部分，独立AOP框架，一般把Aspect和Spimg框架一起使用，进行AOP操作。
    2. 基于Aspect实现AOP操作
    3. 引入依赖
    4. 切入点表达式
    (1) 切入点表达式作用:知道对哪个类里面的哪个方法进行增强。
    (2)语法结构:
    execution([权限修饰符] [返回类型] [类全路径][方法名称]([参数列表]) )
    例1:对com.lele.dao.BookDao里的add方法进行增强
    execution(* com.lele.dao.BookDao.add(..))
    
    例2:对com.lele.dao.BookDao里的所有方法进行增强
    execution(* com.lele.dao.BookDao.*(..))
    
    例3:对com.lele.dao包中的所有类中的所有方法进行增强
    execution(* com.lele.dao.*.*(..))
    ```

- 基于注解实现

    ```java
    // config
    @Configuration
    @ComponentScan(basePackages = {"com.lele"})
    @EnableAspectJAutoProxy(proxyTargetClass = true )
    public class SpringConfig {
    }
    
    //被增强类
    @Component
    public class User {
        public void add(){
            System.out.println("add...");
        }
    }
    
    //增强类
    @Component
    @Aspect //生成代理对象
    @Order(2) // 如果有多个增强类, Order注解判断优先级
    public class UserProxy {
    
        @Pointcut(value = "execution(* lele.aopanno.*.*(..))")
        public void pointDemo(){
        }
        //前置通知
        //@Before注解作为前置通知
        @Before(value = "pointDemo()")
        public void before(){
            System.out.println("UserProxy before..");
        }
        //最终通知
        @After(value = "pointDemo()")
        public void after(){
            System.out.println("after..");
        }
        //后置通知
        @AfterReturning(value = "pointDemo()")
        public void AfterReturning(){
            System.out.println("AfterReturning..");
        }
        //异常通知
        @AfterThrowing(value = "pointDemo()")
        public void AfterThrowing(){
            System.out.println("AfterThrowing..");
        }
        //环绕通知
        @Around(value = "pointDemo()")
        public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable{
            System.out.println("环绕之前..");
    
            proceedingJoinPoint.proceed();
    
            System.out.println("环绕之后..");
        }
    }
    
    //测试结果
    output:
    
    PersonProxy before
    环绕之前..
    UserProxy before..
    add...
    环绕之后..
    after..
    AfterReturning..
    ```

    



