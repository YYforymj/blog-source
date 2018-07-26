---
title: java基础02
date: 2017-04-01 14:32:03
categories: java
---
> 以下内容为传智博客的《java基础入门》一书的读书笔记

<!-- more -->

## 第四章
* final修饰的类不可继承，方法不可重写，标识符为常量
* 包含抽象方法的类必须为抽象类，接口中方法默认使用public abstract修饰，变量默认使用public static final修饰。抽象类中可以有非抽象方法，但必须包含抽象方法。
* 接口可以通过匿名内部类的方式直接实例化，且接口支持多重继承，java的多重继承也是通过接口实现的（java的类不支持多重继承）
* instanceof 用于判断一个对象是否是一个接口或一个类的实例或子实例
* 匿名内部类的实例
```
interface Animal{
    void shout();
}
public class cat{
    public static void main(String[] args){
        animalShout(new Animal(){
           public void shout(){
                System.out.println("miao");
           }
      })
   }
    public static void animalShout(Animal an){
        an.shout();
   }
}
```
* 异常
![Exception](\images\java02\Exception.png)
Error是程序无法处理的错误，比如OutOfMemoryError、ThreadDeath等。这些异常发生时，Java虚拟机（JVM）一般会选择线程终止。 
Exception是程序本身可以处理的异常，这种异常分两大类运行时异常和非运行时异常。程序中应当尽可能去处理这些异常。
* 异常的捕获：使用try catch捕获
finally的理解：try...catch后无论发生异常还是return退出，finally块中的语句都会执行，如果如turn返回了值而finally也返回了值，则finally中的返回值覆盖之前的返回值，之前的返回值无效
* throw与throws的区别：throws用于抛出方法产生的异常，在方法声明处使用；throw用于抛出自定义异常，在方法内使用
* java常用包：
java.lang包含String,Math,System,Thread等核心类；
java.util包含工具类，集合类
* 访问控制：
public：任意类访问（公共访问）
protected：只能被同包或不同包的子类访问（子类访问）
default：只能被同包的类访问（包访问）
private：只能被该类其他成员访问（类访问）
* 静态导包：一个包中某个类的方法全部为static，可以静态导入。例子：import static packagename;
* 自定义异常 
```
public class DivideByMinusException extends Exception{
	public DivideByMinusException(){
		super();
	}
	public DivideByMinusException(String msg){
		super(msg);
	}
}
public class ExceptionDemo{
	public static void main(String[] args){
		int res = divide(4,-2);
		System.out.println(res);
	}
	public static int divide(int a,int b){
		if(b < 0){
			throw new DivideByMinusException("Divide by minus number!");
		}
		return a/b;
	}
}
```

## 第五章
* 多线程的两种实现方式：
1. 继承Thread类，覆写run()方法，在run()中实现线程
2. 实现Runnable接口
* 继承Thread类实现多线程，需要注意的是，run()方法不会启动新县城，start()方法才会启动
```
public class Demo1{
	public static void main(String[] args){
		MyThread my = new MyThread();
		my.start();
		while(true){
			System.out.println("main thread is running");
		}
	}
}
class MyThread extends Thread{
	public void run(){
		while(true){
			System.out.println("my thread is running");
		}
	}
}
```
* 实现Runnable接口创建多线程
```
public class Demo2{
	public static void main(String[] args){
		MyThread  my = new MyThread();
		Thread thread = new Thread(my);
		thread.start();
		while(true){
			System.out.println("main thread is running");
		}
	}
}
class MyThread implements Runnable{
	public void run(){
		while(true){
			System.out.println("my thread is running");
		}
	}
}
```
* 两种方式的比较：
由于java的单继承特性，继承Thread类后无法继承其他类，故通常用实现Runnable接口方法；使用Runnable接口可以使得多线程共享资源（在Runnable实现类中实现共享资源）；
* 后台线程：
通过setDemon(true)实现，且需要在start()方法调用前使用；isDemon()可以判断后台线程；前台进程全部结束后，JVM会在一定时间内杀死全部后台进程
* 线程的五种状态：新建、就绪、运行、阻塞、死亡
死亡状态的条件：run()执行完成或者发生Exception/Error；
* 线程优先级分为1-10级，数字越大优先级越高（MAX:10;NORM:5;MIN:1;）；main方法的优先级默认为5；Thread类的setPriority(int Priority)可以设置优先级；
* sleep()方法执行完成后线程为就绪状态；yield()方法会将线程置于就绪状态；join()实现线程插队，即线程一调用线程而的join()方法，则线程一就会在线程二执行结束后执行；wait()方法使当前线程房企同步锁并等待，知道其他进程进入该锁并唤醒；notify()唤醒第一个wait()的线程；notifyAll()唤醒全部wait()的线程
* 同步关键字synchronized：同步代码块的锁是任意对象，该对象要提前声明；同步方法的锁是调用该方法的对象，静态同步方法的锁是该方法所在类的类对象（Class），可以用“类名.class”获取
* 死锁产生的四个必要条件
1. 互斥条件：一个资源每次只能被一个线程使用
2. 请求与保持条件：一个线程因请求资源而被阻塞时，对于该线程已经获得的资源不释放
3. 不剥夺条件：线程已经获得的资源在未使用完成前不会被强制剥夺
4. 循环条件：若干进程间形成头尾循环等待资源关系
死锁实例
```
class Deadlocker{  
	int field_1;  
	private Object lock_1 = new int[1];  
	int field_2;  
	private Object lock_2 = new int[1];  
	
	public void method1(int value){  
		“synchronized” (lock_1){  
			“synchronized” (lock_2){  
			field_1 = 0; field_2 = 0;  
		   }  
		}
	  }  
  
	public void method2(int value){  
		“synchronized” (lock_2){  
			“synchronized” (lock_1){  
			field_1 = 0; field_2 = 0;  
			}  
		}  
	}  
} 
```
上例中考虑如下现象
一个线程（ThreadA）调用method1(),ThreadA在lock_1上同步，但允许被抢先执行。
另一个线程（ThreadB）开始执行,ThreadB调用method2()。
ThreadB获得lock_2，继续执行，企图获得lock_1,但ThreadB不能获得lock_1，因为ThreadA占有lock_1,ThreadB阻塞。
现在轮到ThreadA继续执行。ThreadA试图获得lock_2，但不能成功，因为lock_2已经被ThreadB占有了,ThreadA和ThreadB都被阻塞，程序死锁。

## 第六章
* StringBuffer没有覆写equals方法，不能用+连接，连接要用append()方法；
* 包装类：使用valueof()返回包装类的值（int或String）；parseInt(Stirng s)将字符串参数作为有符号的十进制整数进行解析；

 