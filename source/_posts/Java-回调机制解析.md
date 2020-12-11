---
title: Java 回调机制解析
date: 2019-05-29 11:51:58
tags: "Java"
categories: "Java"
---

# 调用概述

​		在讲回调之前需要对调用方式有个概念。函数调用方式分为两种，分别是**同步调用**和**异步调用**。

​		同步调用是最基本的调用方式：**类A的方法a()调用类B的方法b()，然后一直等待b()方法执行完毕后，a()方法再继续往下执行。**这种调用方式是顺序执行的，适用于需要顺序执行的情况。

​		异步调用为了并行执行程序而出现的调用方式：**类A的方法a()通过新起线程的方式调用类B的方法b()，然后代码接着直接往下执行，与b()方法并行执行。**这种方式适用于代码需要同时执行或b()方法执行时间过长，防止阻塞的情况。

​		回调和一般调用的区别在于：**在被调用者类B的方法b()执行完毕后，再去调用类A的方法c()，进行类似于一个通知的操作。**

----



# 回调概述

​		回调：**类A的a()方法调用类B的b()方法，当类B的b()方法的执行完成后又去调用类A里的c()方法，进行通知。**是一种双向的调用方式，类似于消息或事件的机制。

​		一般情况下，回调分两种，分别是**同步回调**和**异步回调**，主要区别在于是否新建线程执行。

## **同步回调**

​		同步，就说明了这是是一种顺序执行的方式，需要等待执行结果出来后，再继续执行下一步。下面用一个计算器的例子来展示说明。

```java
public class CallBack
{
     public static void main(String []args)
     {
        int a = 10452;
        int b = 423;
        //实例化计算器Calculator类
        Calculator calculator = new Calculator(a,b);
        //调用计算器calculator的计算函数
        calculator.calculation();
        //控制台输出
        System.out.println("/========================/");
     }
}

//计算器类
class Calculator
{
    private int a,b;

    public Calculator(int a, int b)
    {
        this.a = a;
        this.b = b;
    }

    public void calculation()
    {
        new Logic().calculationLogic(a, b, Calculator.this);
    }

    //用于接收回调结果，并打印输出
    public void calculationResult(int a, int b, int result)
    {
        //控制台输出计算
        System.out.println(a + " + " + b + " = " + result);
    }
}

//计算的具体逻辑类
class Logic
{
    //计算的具体逻辑(计算方式)
    public void calculationLogic(int a, int b, Calculator calculator)
    {
        int result = a + b;
        //让线程等待5秒，模拟需要执行的时间
        try
        { 
            Thread.sleep(5 * 1000); 
        } catch (InterruptedException e)
        { 
            e.printStackTrace(); 
        } 
        //利用传进来的对象,回调计算结果.
        calculator.calculationResult(a, b, result);
    }
}
```

```
运行结果： 
10452 + 423 = 10875 
/========================/
```



------

## **异步回调**

​		异步，就说明了这是是一种并行执行的方式，不需要等待执行结果出来后再继续执行，而是可以同步进行，执行结果出来后，再回调进行通知。下面还是用一个计算器的例子来展示说明。

```java
public class CallBack
{
     public static void main(String []args)
     {
        int a = 10452;
        int b = 423;
        //实例化计算器Calculator类
        Calculator calculator = new Calculator(a,b);
        //调用计算器calculator的计算函数
        calculator.calculation();
        //控制台输出
        System.out.println("/========================/");
     }
}

//计算的具体逻辑类
class Logic
{
    //计算的具体逻辑(计算方式)
    public void calculationLogic(int a, int b, Calculator calculator)
    {
        int result = a + b;
        //让线程等待5秒
        try
        { 
            Thread.sleep(5 * 1000); 
        } catch (InterruptedException e)
        { 
            e.printStackTrace(); 
        } 
        //利用传进来的对象,回调计算结果.
        calculator.calculationResult(a, b, result);
    }
}

//计算器类
class Calculator
{
    private int a,b;

    public Calculator(int a, int b)
    {
        this.a = a;
        this.b = b;
    }

    public void calculation()
    {
        //开启另一个子线程
        new Thread(new Runnable()
        { 
            public void run()
            { 
                new Logic().calculationLogic(a, b, Calculator.this);
            } 
        }).start(); 
    }

  	//用于接收回调结果，并打印输出
    public void calculationResult(int a, int b, int result)
    {
        //控制台输出计算结果
        System.out.println(a + " + " + b + " = " + result);
    }
}
```

```
运行结果： 
/========================/ 
10452 + 423 = 10875
```

​	你会发现，输出”/====/”明明是放在代码的最后，结果却先执行输出了，这是因为开了另一个线程，而异步回调和同步回调最大的区别就是异步回调里新建了一个线程。异步回调常见于请求服务器数据，当取到数据时,会进行回调。

------

# **扩展知识**

​	另一种回调方式，主要是为解决当回调的逻辑不明确时，该如何处理回调逻辑。解决方法：**传入实现的接口对象，在对象里实现具体的回调方法逻辑。**

​		下面的还是用计算器的例子，比如不一定用计算器进行加法运算，也有可能进行乘法运算，下面是同步回调。

```java
public class CallBack
{
     public static void main(String []args)
     {
        int a = 10452;
        int b = 423;
        //实例化计算器Calculator类,并传一个匿名的Logic对象，重写calculationLogic()方法
        Calculator calculator = new Calculator(new Logic()
        {
            //重写计算逻辑函数calculationLogic,实现具体计算逻辑
            public void calculationLogic(int a, int b){
                int result = a * b;
                System.out.println(a + " * " + b + " = " + result);
            }
        });
        //调用计算器calculator的计算函数calculation
        calculator.calculation(a, b);
        //控制台输出
        System.out.println("/========================/");
     }
}

//计算的逻辑回调接口
interface Logic
{
    //计算的逻辑回调函数
     public void calculationLogic(int a, int b);
}

//计算器类
class Calculator
{
    private Logic logic;

    public Calculator(Logic logic)
    {
        this.logic = logic;
    }

    public void calculation(int a, int b)
    {
        //调用logic对象里的计算逻辑函数calculationLogic
        logic.calculationLogic(a, b);
    }
}
```

```
运行结果： 
10452 * 423 = 4421196 
/========================/
```

​		异步回调也同理，在Calculator类的calculation()函数里新建一个线程就行了，这里就不举例了。

​		既然了解了这种回调方法，那么继续深入这种方式具体能在什么现实的场景下使用呢？现在我提一个需求：给定一个数字，再通过网络请求获取一个数字，然后把这两个数字进行加减或乘除（即实现逻辑不明确），同时不能阻塞线程。没错，通过上面传入实现接口对象的异步回调方式去实现。下面是代码展示。

```java
public class Main {

    public static void main(String[] args) {
        int a = 3;
        //实例化计算器Calculator类,并传一个匿名的Logic对象，重写calculationLogic()方法
        new Calculator(a, new Calculator.Logic()
        {
            //重写计算逻辑函数calculationLogic,实现具体计算逻辑
            public void calculationLogic(int a, int b){
                int result = a * b;
                System.out.println(a + " * " + b + " = " + result);
            }
        }).start();

        //控制台输出
        System.out.println("/========================/");
    }
}

//计算器类
class Calculator
{
    private int a;
    private Logic logic;

    public Calculator(int a, Logic logic)
    {
        this.a = a;
        this.logic = logic;
    }

    public Calculator start()
    {
        //开启另一个线程
        new Thread(new Runnable()
        {
            public void run()
            {
                //让线程等待5秒,模拟请求网络的耗时
                try
                {
                    Thread.sleep(5 * 1000);
                } catch (InterruptedException e)
                {
                    e.printStackTrace();
                }

                int b = 10;//模拟请求到的数据
                logic.calculationLogic(a, b);
            }
        }).start();

        return this;//返回当前对象
    }

    //计算的逻辑回调接口
    interface Logic
    {
        //计算的逻辑回调函数
        public void calculationLogic(int a, int b);
    }
}
```

```
运行结果： 
/========================/
3 * 10 = 30
```

