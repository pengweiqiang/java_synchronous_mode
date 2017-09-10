为何要使用同步？ 
    java允许多线程并发控制，当多个线程同时操作一个可共享的资源变量时（如数据的增删改查）， 
    将会导致数据不准确，相互之间产生冲突，因此加入同步锁以避免在该线程没有完成操作之前，被其他线程的调用， 
    从而保证了该变量的唯一性和准确性。
  
 
 
一、实例
       举个例子，如果一个银行账户同时被两个线程操作，一个取100块，一个存钱100块。假设账户原本有0块，如果取钱线程和存钱线程同时发生，会出现什么结果呢？取钱不成功，账户余额是100.取钱成功了，账户余额是0。但哪个余额对应哪个呢？很难说清楚，因此多线程的同步问题就应运而生。
 
二、不使用同步方法时

public class Bank {

    private int count =0;//账户余额
    
    //存钱
    public  void addMoney(int money){
        count +=money;
        System.out.println(System.currentTimeMillis()+"存进："+money);
    }
    
    //取钱
    public  void subMoney(int money){
        if(count-money < 0){
            System.out.println("余额不足");
            return;
        }
        count -=money;
        System.out.println(+System.currentTimeMillis()+"取出："+money);
    }
    
    //查询
    public void lookMoney(){
        System.out.println("账户余额："+count);
    }
}


package threadTest;


public class SyncThreadTest {

    public static void main(String args[]){
        final Bank bank=new Bank();
        
        Thread tadd=new Thread(new Runnable() {
            
            @Override
            public void run() {
                // TODO Auto-generated method stub
                while(true){
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                    bank.addMoney(100);
                    bank.lookMoney();
                    System.out.println("\n");
                    
                }
            }
        });
        
        Thread tsub = new Thread(new Runnable() {
            
            @Override
            public void run() {
                // TODO Auto-generated method stub
                while(true){
                    bank.subMoney(100);
                    bank.lookMoney();
                    System.out.println("\n");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }    
                }
            }
        });
        tsub.start();
        
        tadd.start();
    }
}

运行结果如下：

1502542307917取出：100
账号余额：100


1502542308917存进:100
1502542308917取出：100
账号余额：0


账号余额：0


1502542309917存进:100
账号余额：0


1502542309917取出：100
账号余额：0

此时出现了非线程安全问题，因为两个线程同时访问一个没有同步的方法，如果这两个线程同时操作业务对象中的实例变量，就有可能出现非线程安全问题。
解决方案：只需要在public void run()前面加synchronized关键词即可。

三、使用同步时的方案<br/>
(1)同步synchronized方法 <br/>
    即有synchronized关键字修饰的方法。 
    由于java的每个对象都有一个内置锁，当用此关键字修饰方法时， 
    内置锁会保护整个方法。在调用该方法前，需要获得内置锁，否则就处于阻塞状态。
    代码如： 
    public synchronized void save(){}
    注： synchronized关键字也可以修饰静态方法，此时如果调用该静态方法，将会锁住整个类


public class Bank {

    private int count =0;//账户余额
    
    //存钱
    public  synchronized void addMoney(int money){
        count +=money;
        System.out.println(System.currentTimeMillis()+"存进："+money);
    }
    
    //取钱
    public  synchronized void subMoney(int money){
        if(count-money < 0){
            System.out.println("余额不足");
            return;
        }
        count -=money;
        System.out.println(+System.currentTimeMillis()+"取出："+money);
    }
    
    //查询
    public void lookMoney(){
        System.out.println("账户余额："+count);
    }
}

运行结果：

余额不足
账号余额：0


1502543814934存进:100
账号余额：100


1502543815934存进:100
账号余额：200


1502543815934取出：100
账号余额：100

这样就实现了线程同步。
 
(2).同步synchronized代码块 <br>
    即有synchronized关键字修饰的语句块。 
    被该关键字修饰的语句块会自动被加上内置锁，从而实现同步 
    代码如： 
    synchronized(object){ 
    }
     注：同步是一种高开销的操作，因此应该尽量减少同步的内容。 
    通常没有必要同步整个方法，使用synchronized代码块同步关键代码即可。 


public class Bank {
    private int count =0;//账户余额
    //存钱
    public  void addMoney(int money){
        synchronized (this) {
            count +=money;
        }
        System.out.println(System.currentTimeMillis()+"存进："+money);
    }
    //取钱
    public   void subMoney(int money){
        synchronized (this) {
            if(count-money < 0){
                System.out.println("余额不足");
                return;
            }
            count -=money;
        }
        System.out.println(+System.currentTimeMillis()+"取出："+money);
    }
    //查询
    public void lookMoney(){
        System.out.println("账户余额："+count);
    }
}


运行结果：

余额不足
账户余额：0


余额不足
账户余额：100


1502544966411存进：100
账户余额：100


1502544967411存进：100
账户余额：100


1502544967411取出：100
账户余额：100


1502544968422取出：100

这样也实现了线程同步，运行效率上来说也比方法同步效率高，同步是一种高开销的操作，因此应该尽量减少同步的内容。通常没有必要同步整个方法，使用synchronized代码块同步关键代码即可。。


(3).使用特殊域变量(volatile)实现线程同步
    a.volatile关键字为域变量的访问提供了一种免锁机制， 
    b.使用volatile修饰域相当于告诉虚拟机该域可能会被其他线程更新， 
    c.因此每次使用该域就要重新计算，而不是使用寄存器中的值 
    d.volatile不会提供任何原子操作，它也不能用来修饰final类型的变量 
 
Bank.java代码如下：

package com.thread.demo;

/**
 * Created by HJS on 2017/8/12.
 */
public class Bank {
    private volatile int count =0;//账户余额
    //存钱
    public  void addMoney(int money){
        synchronized (this) {
            count +=money;
        }
        System.out.println(System.currentTimeMillis()+"存进："+money);
    }
    //取钱
    public   void subMoney(int money){
        synchronized (this) {
            if(count-money < 0){
                System.out.println("余额不足");
                return;
            }
            count -=money;
        }
        System.out.println(+System.currentTimeMillis()+"取出："+money);
    }
    //查询
    public void lookMoney(){
        System.out.println("账户余额："+count);
    }
}

运行结果：

余额不足
账户余额：0


余额不足
账户余额：100


1502546287474存进：100
账户余额：100


1502546288474存进：100
1502546288474取出：100
账户余额：100

        此时，顺序又乱了，说明同步又出现了问题，因为volatile不能保证原子操作导致的，因此volatile不能代替synchronized。此外volatile会组织编译器对代码优化，因此能不使用它就不适用它吧。它的原理是每次要线程要访问volatile修饰的变量时都是从内存中读取，而不是存缓存当中读取，因此每个线程访问到的变量值都是一样的。这样就保证了同步。
        
（4）使用重入锁实现线程同步

    在JavaSE5.0中新增了一个java.util.concurrent包来支持同步。ReentrantLock类是可重入、互斥、实现了Lock接口的锁， 它与使用synchronized方法和快具有相同的基本行为和语义，并且扩展了其能力。

     ReenreantLock类的常用方法有：

         ReentrantLock() : 创建一个ReentrantLock实例 

         lock() : 获得锁 

         unlock() : 释放锁 

    注：ReentrantLock()还有一个可以创建公平锁的构造方法，但由于能大幅度降低程序运行效率，不推荐使用 

Bank.java代码修改如下：

public class Bank {

    private  int count = 0;// 账户余额
    //需要声明这个锁
    private Lock lock = new ReentrantLock();

    // 存钱
    public void addMoney(int money) {
        lock.lock();//上锁
        try{
            count += money;
            System.out.println(System.currentTimeMillis() + "存进：" + money);

        }finally{
            lock.unlock();//解锁
        }
    }
    // 取钱
    public void subMoney(int money) {
        lock.lock();
        try{

            if (count - money < 0) {
                System.out.println("余额不足");
                return;
            }
            count -= money;
            System.out.println(+System.currentTimeMillis() + "取出：" + money);
        }finally{
            lock.unlock();
        }
    }
    // 查询
    public void lookMoney() {
        System.out.println("账户余额：" + count);
    }
}

运行效果：

余额不足
账户余额：0


1502547439892存进：100
账户余额：100


1502547440892存进：100
账户余额：200


1502547440892取出：100
账户余额：100

  注：关于Lock对象和synchronized关键字的选择： 
        a.最好两个都不用，使用一种java.util.concurrent包提供的机制， 
            能够帮助用户处理所有与锁相关的代码。 
        b.如果synchronized关键字能满足用户的需求，就用synchronized，因为它能简化代码 
        c.如果需要更高级的功能，就用ReentrantLock类，此时要注意及时释放锁，否则会出现死锁，通常在finally代码释放锁 。
 
 
 (5).使用局部变量ThreadLocal实现线程同步

public class Bank {

    private static ThreadLocal<Integer> count = new ThreadLocal<Integer>(){
        @Override
        protected Integer initialValue() {
            // TODO Auto-generated method stub
            return 0;
        }

    };
    // 存钱
    public void addMoney(int money) {
        count.set(count.get()+money);
        System.out.println(System.currentTimeMillis() + "存进：" + money);

    }

    // 取钱
    public void subMoney(int money) {
        if (count.get() - money < 0) {
            System.out.println("余额不足");
            return;
        }
        count.set(count.get()- money);
        System.out.println(+System.currentTimeMillis() + "取出：" + money);
    }

    // 查询
    public void lookMoney() {
        System.out.println("账户余额：" + count.get());
    }
}

运行效果：

余额不足
账户余额：0


余额不足
1502547748383存进：100
账户余额：100
账户余额：0




余额不足
账户余额：0


1502547749383存进：100
账户余额：200

看了运行效果，一开始一头雾水，怎么只让存，不让取啊？看看ThreadLocal的原理：
如果使用ThreadLocal管理变量，则每一个使用该变量的线程都获得该变量的副本，副本之间相互独立，这样每一个线程都可以随意修改自己的变量副本，而不会对其他线程产生影响。现在明白了吧，原来每个线程运行的都是一个副本，也就是说存钱和取钱是两个账户，知识名字相同而已。所以就会发生上面的效果。
    ThreadLocal 类的常用方法
     ThreadLocal() : 创建一个线程本地变量 
    get() : 返回此线程局部变量的当前线程副本中的值 
    initialValue() : 返回此线程局部变量的当前线程的"初始值" 
    set(T value) : 将此线程局部变量的当前线程副本中的值设置为value
 
注：ThreadLocal与同步机制 
        a.ThreadLocal与同步机制都是为了解决多线程中相同变量的访问冲突问题。 
        b.前者采用以"空间换时间"的方法，后者采用以"时间换空间"的方式 

（6）wait/notifyAll 方式

wait/notifyAll方式跟ReentrantLock/Condition方式的原理是一样的。
Java中每个对象都拥有一个内置锁，在内置锁中调用wait，notify方法相当于调用锁的Condition条件对象的await和signalAll方法。
