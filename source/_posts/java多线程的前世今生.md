---
title: java多线程的前世今生
top: true
cover: false
toc: true
mathjax: true
date: 2021-07-12 15:45:51
password:
summary:
categories: 编程语言
tags: 
	- java
	- 并发编程
---

## 前言

一个多月没写过博文了，不是没有**值得记录的故事**，确是静不下心。网申在即，靠目前的积淀，知识地图中缺了一大块，索性打消了网申的念头。但不免有些落寞，像极了**落跑的小橘子**。

这个`时代越来越卷了`，但也正因如此，我才更信奉积淀的魅力，这也正是我整理心态，继续出发的缘由。

这不是第一次学习多线程了，在自学时，培训班上都接触过多线程，但都只是浅尝辄止，这次咬定青山不放松，终于**顿悟了一些关键的知识**，恍然间柳暗花明，便有了这篇文章。

------

## 多线程——入门

在操作系统中，我们了解到，**多进程图像**是操作系统最核心，最基础的图像。而用户通过系统调用的方式间接与操作系统交互。想象这样一种场景，我们一边放着音乐，一边浏览网页——

![image-20210327092252968](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210327092252968.png)

这就是触目可及的多进程图像，显然，这两个进程之间使用的是不同的资源，不妨假设我的处理器是单核的，每一时刻只能处理一个进程，那`CPU就要为两个进程分配时间片`，然后根据调度算法切换进程，串行执行。

如何切换进程呢？首先要理解进程是什么——

![image-20210619000547145](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210619000547145.png)

也就是说，进程就是：**指令执行序列+资源**，因此直观的切换进程就是切换进程的这两部分，但资源的切换是非常浪费时间的，能不能`只切换指令执行序列`呢？聊到这，小伙伴就有了这样的猜想：指令执行序列与线程有什么关系呢？

没错，**指令执行序列就是线程集**。我们深入到进程中找找，就会发现线程集没有我们想象中的那么遥不可及——

![image-20210327094410019](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210327094410019.png)

下载听歌两不误，如此丝滑的切换，都得益于**线程的并发应用**。也就是多线程咯。到这里，故事才真正的拉开序幕——

------

### 1.进程与线程

前面提到的都是直观的印象，这里由表及内地聊聊进程与线程——

- `进程： `进程是**上下文切换之间的程序执行的部分**，而所谓上下文，即是`CPU与寄存器`。通俗地说，也就是说进程执行的环境。在这个环境下运行着的程序即是进程。
- `线程： `线程是**轻量级的进程**，这里的轻量级如何理解呢？`线程集`共享了进程的`上下文环境，地址空间，更为细小的CPU时间片`。

对于如何更好地理解进程与线程之间的关系，有大佬已经讲的非常形象了，我们不妨站在巨人的肩膀上——

- [x] [将进程比作火车，线程比作车厢](https://www.zhihu.com/question/25532384)

------

### 2.所谓多线程

在讲述多线程之前，非常有必要搞懂这几个概念——

- `并发`：一些任务都处于可运行的状态，但只有一个任务可以被调度并执行，而具体是哪一个任务被处理仍是不确定的。

- `串行与并行`

  1. 多任务中**某时刻**只能有一个任务被运行，单核下皆为串行；
  2. 某时刻可以有多个任务同时被执行，在多核计算机上可以实现并行；

- `阻塞与非阻塞`：

  1. 以文件读取为例，应用层会调用系统内核中的IO接口，如open，close等，在请求发出以后，如果调用的是**阻塞型IO**，应用层被挂起，一直处于等待数据返回的状态。直到从磁盘上读取数据并返回给应用层后，应用层采用获取到的数据进行下一步操作；
  2. 如果应用层调用的是**非阻塞I/O**，那么调用后，系统内核会立即返回（虽然还没有文件内容的数据），应用层并不会被挂起，它可以做其他任意它想做的操作。

  也就是说阻塞与非阻塞的区别在于**发出请求后等待数据返回时的状态**。

- `同步与异步`：

  - 阻塞与非阻塞决定了应用层等待数据返回的状态，而**数据如何从系统内核返回**呢？返回的方式取决于同步与异步——
  - 对于同步型的调用，应用层会不**断问询系统内核**，但它的动作取决于是否阻塞，数据返回给应用层后，应用层继续做其他的事情；
  - 对于异步型的调用，应用层**不关心数据何时返回**，此时它的动作同样却绝育是否阻塞，系统内核会主动通知应用层数据已经读取完毕并返回。

------

好啦，啰嗦了这么多，也该祭出这张图了——

![image-20210327104345505](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210327104345505.png)

用语言描述就是：**多线程无非是多个线程处于并发的状态**。

片头就这么多了，暂且告一段落。如果哪天灵光一现，也会再修修补补。

> 日拱一卒，功不唐捐。

------

## 关于下文

借鉴王一波前辈的书名，下面的篇幅按照多线程的重要程度渐进地划分为三个阶段——

- `青铜时代`：主要描述线程的基本知识，也是最不可或缺的一部分，此外，由此衍生的静态代理，lambda表达式也可先入门一手；
- `白银时代`：这一阶段是多线程并发的问题与基本解决方案，同样也是应用与面试的一大热门。
- `黄金时代`：简言之，这一部分是高端操作，也是面试造火箭的宠儿，但其中的机制却并不新鲜，反而有点拾操作系统思想牙髓的意思。可见，最基本，最通用的反而具有最长久的生命力，

------

## 多线程——青铜时代

### 1.创建的三种方式

在java中，创建线程有3种方式，分别是继承Thread类，实现Runnable接口，实现Callable接口。而实现Runnable接口只能得到任务，**任务还需要线程驱动才能执行**、

#### 实现Runnable接口

![image-20210327130931773](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210327130931773.png)

首先实现Runnable接口得到Runnable实例。

```java
public class CreateThread2 implements Runnable{
   
    @Override
    public void run() {
      //do something
    }
}
```

使用Runnable实例再创建一个Thread实例，调用Thread实例的start()开启一个线程——

```java
public static void main(String[] args) {
    //创建实现类
    CreateThread2 t = new CreateThread2();
    //同样调用Thread来开启线程，最终是使用runnable接口的代理来调用
    new Thread(t).start();
}
```

#### 实现Callable接口

与Runnable相比，Callable接口具有返回值。返回值用**Future<V>**封装——

```java
/**
 * 使用Callable接口实现图片的下载
 *
 * callable的好处——
 * 1. 有返回值
 */
public class CreateThread3 implements Callable<Boolean> {
    private String url;
    private String name;

    public CreateThread3(String url, String name) {
        this.url = url;
        this.name = name;
    }

    @Override
    public Boolean call() throws Exception {

        try {
            WebDownload webDownload = new WebDownload();
            webDownload.download(url,name);
            System.out.println(name+"下载成功");
        } catch (MalformedURLException e) {
            e.printStackTrace();
            System.out.println("图片下载异常");
        }
        return true;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CreateThread3 t1 = new CreateThread3("https://img95.699pic.com/photo/40011/0709.jpg_wh860.jpg","E:/1.png");
        CreateThread3 t2 = new CreateThread3("https://static.runoob.com/images/demo/demo4.jpg","E:/2.png");
        CreateThread3 t3 = new CreateThread3("https://seopic.699pic.com/photo/50034/6414.jpg_wh1200.jpg","E:/3.png");

        //创建固定线程数的线程池
        ExecutorService service = Executors.newFixedThreadPool(3);

        //提交执行
        Future<Boolean> r1 = service.submit(t1);
        Future<Boolean> r2 = service.submit(t2);
        Future<Boolean> r3 = service.submit(t3);

        //获取结果
        Boolean res1 = r1.get();
        Boolean res2 = r2.get();
        Boolean res3 = r3.get();

        //关闭服务
        service.shutdownNow();
    }
}

class WebDownload{
    public void download(String url,String name) throws IOException {
        FileUtils.copyURLToFile(new URL(url),new File(name));
    }
}
```

#### 继承Thread类

继承Thread类同样需要重写run方法，但run方法却仅仅只是Thread类的冰山一角，Thread类非常庞大，它包含了`线程的状态，分类，并发安全`，不一而足：

![image-20210327132338149](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210327132338149.png)

来看代码实现——

```java
package createThread;

public class CreateThread1 extends Thread{
    /**
     * run方法线程体
     */
    @Override
    public void run() {
        super.run();
        for (int i=0;i<20;i++){
            System.out.println("五味杂陈");
        }
    }

    /**
     * 主线程方法体
     * @param args
     */
    public static void main(String[] args) {
        CreateThread1 t = new CreateThread1();
        //可以看到调用start方法以后，主线程与run方法线程交替执行，总体先执行main主线程，也许是因为优先级较高！
        //假如不是多线程，会怎样执行？
        //那就应该在主线程中调用run的线程，在run线程执行完后再回到主线程，绝不会发生交替！
        //线程的调度完全由CPU决定
        t.start();
        //可能会被提前终止
        t.interrupt();
        for (int i=0;i<200;i++){
            System.out.println("陈杂味五");
        }
    }
}
```

当调用start以后，虚拟机将线程放入就绪队列等待被调度，线程被调度后则执行run方法。

#### Runnable VS Thread

实现接口会更好一些，因为：

1. java是`单继承，可扩展性不好`，继承了Thread类就无法再继承其他类，而接口可以多实现；
2. `Thread类这么臃肿`，如果只是执行简单的线程任务，则继承Thread类显得大而无用。

------

### 2.线程的状态与属性

#### 开局一张图

![线程的状态](C:\Users\zyz\Desktop\每日思维导图\线程的状态.png)

关于状态的琐碎知识，我就不展开了，**思维导图描述的更系统也更简洁这样记忆的效果更好**，针对状态的切换，在网上找到一张很细致的图片——
![image-20210327134912906](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210327134912906.png)

来看切换内部的机制——

![image-20210327141723664](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210327141723664.png)

#### 一点小贴士

在这里，我们来解释一些关于线程切换与分类中，含糊不清的知识点：

1. `进程与线程的状态基本一致`，一家之言；
2. `阻塞是一种状态，挂起是一种行为`，挂起的结果是阻塞；
3. 当run方法执行完后，线程则退出，变为死亡状态。而使用执行器，需要手动的关闭服务，从而关闭线程，关闭服务`shutDownNow`的底层会`调用interrupt方法`，该方法不一定是即时的，`它会使线程在安全的时候退出`,这同时也是中断的用途。
4. `wait会释放对象的锁，而sleep并不会`。也就是说线程调用wait进入阻塞状态，使用notify唤醒线程，而后进入就绪队列等待。而sleep则可以直接执行。

### 3.线程的基本机制

在上面的内容中，我们突兀地使用了**很多看起来没有支撑的知识点**。我想小伙伴们也同样会困惑，对象锁是如何实现的？它的机制是怎样的？线程这么多的状态，我们都要耳提面命的测试吗？

对于这些问题，java中的多线程给出了一个基本的机制来解决——

#### 对象锁

在JVM中，对象在内存中被分成了三个区域——

1. **对象头**
   1. `Mark_word(标记字段)`，默认存储`对象的锁标志位`，hashcode,分代年龄。它会根据锁的状态复用它的存储空间，也就是说，运行期间Mark_word中的信息会随着锁对象的变化而变化；
   2. `Klass_point(类型指针)`：即对象指向它的类元数据的指针，虚拟机通过这个指针来确定对象属于哪个类。
2. **实例信息**：主要存放类的数据信息，父类的数据信息；
3. **对齐填充**：即任何一个对象，都会被按位填充，即便是空对象也是如此。

线程拥有对象的锁标志位，即表示线程可以访问该对象。从而可以通过对对象锁的操作来实现并发安全。

#### 执行器

如果程序员直接管理线程的生命周期，未免舍本逐末，太过繁琐。java为我们提供了大管家——***Executor***。

**Exector会将线程的开启与运行机制分离开**，简化程序员的操作。它是是JUC包下的一个类，提供了三种可选的Executor:

- `CachedThreadPool`：一个任务创建一个线程；
- `FixedThreadPool`：所有任务只能使⽤用固定⼤小的线程；
- `SingleThreadExecutor`：相当于大小为 1 的 FixedThreadPool。

简要看下它的使用——

```java
public static void main(String[] args) {
    //创建线程池
    ExecutorService executorService=Executors.newCachedThreadPool();
    //执行
    for (inti=0; i<5; i++) {
        executorService.execute(new MyRunnable());    
    }
    //关闭服务
    executorService.shutdown();}

```

------

## 多线程——番外篇

### 1.静态代理

> 首先，考虑有这样一种需求，项目经理要求在所有类的每一个方法运行前后加上日志，但又要求不能对已有代码做改动，怎样解决呢？

我们可以创建一个代理对象，让它和目标对象实现同样的接口，这样就可以借代理对象之手调用目标对象的方法了。此外，我们还需要为这个代理对象加上日志方法，问题就迎刃而解了。

从这段描述中，我们试着提取静态代理的关键：

- **代理对象=目标对象+加强代码**；
- 代理对象和目标对象需要`实现同一个接口`；

那小伙伴也许会困惑，咋们不是在学习多线程吗，难道多线程中使用到了静态代理？没错，Thread类就可以看做是Runnable接口实现类的静态代理，通过下面的小例子我们来巩固一下——

```java
public class StaticProxy {
    public static void main(String[] args) {

        //使用lambda来开启线程
        new Thread(()->System.out.println("我爱你")).start();
        //类比着看，说明线程底层使用的即是静态代理
        new WeddingCompany(new You()).happyMarry();

        //将真实对象作为代理对象的参数传入
        WeddingCompany weddingCompany = new WeddingCompany(new You());
        weddingCompany.happyMarry();
    }
}

/**
 *都需要实现的顶层接口
 */
interface Marry{
    void happyMarry();
}

/**
 * 真实角色
 */
class You implements Marry {

    @Override
    public void happyMarry() {
        System.out.println("洞房花烛夜");
    }
}

/**
 * 代理角色，代理角色能做的事情更多
 */
class WeddingCompany implements Marry {
    //这里是关键
    private Marry target;

    public WeddingCompany(Marry target) {
        this.target = target;
    }

    @Override
    public void happyMarry() {
        before();
        this.target.happyMarry();//这里是真实对象做的事情
        after();
    }

    private void after() {
        System.out.println("结婚之后，收尾款");
    }

    private void before() {
        System.out.println("结婚之前，布置现场");
    }
}


```

### 2.Lambda表达式

首先通过下面的小例子了解下lambda的写法——

```java
ExecutorService service = Executors.ExecutorService();

service.submit( () -> {
    System.out.println("肚子有点饿");
})
```

而这里的方法参数实际上应该是实现Runnable接口的任务，也就是说，**lambda表达式本身就是接口的实现**。它`扩展了匿名内部类`，使得写法更加简洁高效。

通过下面的实例来看下lambda的成长历程：

```java

```

------

### 3.java内存模型

java试图屏蔽不同硬件与操作系统之间的内存访问差异，来实现java程序在各种平台都能运行的一致性数据访问。

#### 工作内存与主内存

我们知道，CPU只负责运算，它将内存中的数据读取到寄存器中，CPU再取得寄存器的地址，最后执行指令。但寄存器(`CPU的零级缓存`)的读写速度比内存的读写速度快很多个数量级，为了缓冲这种速度差，引入了`高级缓存`。

但加入高级缓存以后，又带来了新的问题：**缓存一致性**。如果多个缓存共享一个主内存中的变量，那么多个缓存中的数据可能不一样。需要引入协议来规范一致性问题：

![image-20210327221223729](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210327221223729.png)

工作内存通常存储在高速缓存和寄存器中，在工作内存中保存了`主内存变量的拷贝`。

#### 内存间的交互操作

java内存模型定义了8个操作来完成主内存和工作内存之间的交互——

![image-20210327221528229](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210327221528229.png)

- `read`：把一个变量的值从主内存传输到工作内存中
- `load`：在 read 之后执行，把 read 得到的值放⼊工作内存的变量副本中
- `use`：把工作内存中⼀个变量的值传递给执行引擎
- `assign`：把一个从执行引擎接收到的值赋给工作内存的变量
- `store`：把工作内存的一个变量的值传送到主内存中
- `write`：在 store 之后执行，把 store 得到的值放入主内存的变量中
- `lock`：作⽤用于主内存的变量

#### 内存模型的三大特性

1. **原子性：**

   上面我们提到了8种对内存的操作，这些操作作用的变量在被volatile修饰时都是原子性的。但`内存模型允许java虚拟机将没有被volatile修饰的64位数据，即long,double，划分为两个32位的数据执行`，即在这种情况下，操作不是原子性的。

   但int型的数据就是原子性的吗？也不尽然，对数据的操作通常不是单一的操作可以完成的，简要的说，count++就分为read,assign,write三步。这三步整合起来并不是原子性的。

   那如何保证原子性呢？`直观的方式就是加锁`咯，将count++的代码块用synchronized修饰。还有其他基于同步框架的方式吗？

   `AtomicInteger`就是JUC框架下的这样一种方式，那它是如何实现的呢？

   ![image-20210327223155578](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210327223155578.png)

   显然，它应用了`CAS的自旋思想`。

2. **可见性：**

   可见性指的是`当一个线程修改了主内存中的值以后，其他线程可以立刻获知这个修改`，java内存模型是通过将变量修改后的值同步回主内存，在变量读取之前刷新主内存中的值来实现的。

   主要有三种实现可见性的方式：

   - `volatile`
   - `synchronized`,对一个变量使用unlock之前，必须将变量的值同步回主内存；
   - `final`,被final修饰的字段，在初始化之后，如果没有发生this逃逸，那么其他线程就能看见final字段的值。

3. **有序性：**

   有序性指的是：`在本线程中观察，所有操作都是有序的。在一个线程中观察另一个线程，所有操作都是无序的`。

   本线程有序还好理解，另一个线程无序就有点摸不着头脑了。

> 无序是因为发生了`指令重排序`，在java内存模型中，允许编译器和处理器对指令进行重排序，重排序的过程不会影响单线程程序的执行，却会影响多线程并发执行的正确性。
>
> volatile关键字通过添加`内存屏障`的方式来禁止指令重排，即重排序时不能把后面的指令放在内存屏障之前。
>
> 当然，也可以通过`sychronized`来保证有序性。

> 日拱一卒，功不唐捐。

------

##  多线程——白银时代

### 1.多线程不安全的案例

多个线程在访问同一个变量时有可能出现并发错误，我们通过下面的例子来演示并发错误，并借此分析并发错误出现的原因及解决方式——

```java
package synchonized;

public class TestMakeup {
    public static void main(String[] args) {
        Thread thread = new Thread(new Makeup(0, "灰菇凉"));
        Thread thread1 = new Thread(new Makeup(1, "白雪公主"));
        thread.start();
        thread1.start();
    }
}

class LipStick{

}

class Mirror{

}

/**
 * 两个女孩分别持有镜子与口红的锁，然后都去等待对方的资源
 */
class Makeup implements Runnable{
    int choice;
    String person;

    //static表示只有一份
    static LipStick lipStick = new LipStick();
    static Mirror mirror = new Mirror();

    public Makeup(int choice, String person) {
        this.choice = choice;
        this.person = person;
    }

    @Override
    public void run() {
        try {
            makeup();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    //互相请求对方所持有的的锁就会导致死锁，死锁的四个必要条件：互斥，请求与保持，不可剥夺，循环等待，
    public void makeup() throws InterruptedException {
        if(choice == 0){
            synchronized (lipStick){
                System.out.println(Thread.currentThread().getName()+"正在使用口红");
                Thread.sleep(1000);//一秒后要去获得镜子
            }
            synchronized (mirror){
                System.out.println(Thread.currentThread().getName()+"正在使用镜子");
            }
            /**死锁——
             * synchronized (lipStick){
                  System.out.println(Thread.currentThread().getName()+"正在使用口红");
                     synchronized (mirror){
                       Thread.sleep(1000);//一秒后要去获得镜子
                       System.out.println(Thread.currentThread().getName()+"正在使用镜子");
                     }
             }
             *
             */
        }else{
            synchronized (mirror){
                System.out.println(Thread.currentThread().getName()+"正在使用镜子");
                Thread.sleep(2000);//2秒后要去获得口红
            }
            synchronized (lipStick){
                System.out.println(Thread.currentThread().getName()+"正在使用口红");
            }
        }
    }
}


```

在上面的例子中，我们看到假如`双方都在持有自己锁的前提下，请求对方的锁`时。可能导致并发错误的表现之一——死锁，这里再来回顾一下形成死锁的必要条件——

- `互斥：`一个资源只能被一个线程使用；
- `循环等待：`死锁的状况不会因为时间改变而改善；
- `请求与持有：`在没有获得请求锁的前提下，不会释放自己持有的锁。
- `不可剥夺：`线程只能主动让出资源，不能强取；

------

那还有什么原因会导致并发错误呢？并发错误的表现有这样几种情况——

1. 访问**共享变量**导致数据不一致；
2. CPU还没有完成所有线程的调度，线程就已经提前结束了；

那么，针对并发错误的原因，我们可以得到解决并发错误的策略——

1. **互斥同步**：对发生更新的数据加锁，即java互斥同步的两种锁机制：synchronized与Lock;
2. **final**: 直接将变量声明为 不可变的，就不存在变量了，这波曲线解决问题；
3. **非阻塞同步**：CAS机制
4. **不同步机制**：需要同步是因为多个线程共享一个变量，那要实现不同步，就让每个线程独享一份变量就好咯，通常解决方式有这么几种：
   1. `栈封闭`
   2. `线程本地存储`
   3. `可重入代码`

这里我们暂且只介绍互斥同步的解决方式，其他策略在黄金时代中会重点描述，现在混个眼熟~

------

### 2.synchronized

#### 1.synchronized的使用场景

- 修饰`代码块`

  ![image-20210328100335007](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210328100335007.png)

- 修饰`实例方法`，也就是对this对象加锁

  ![image-20210328100502317](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210328100502317.png)

- 修饰`静态方法`，即对class对象加锁

  ​	![image-20210328100915411](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210328100915411.png)

#### 2.synchronized的基本机制

我们之前谈到java内存模型有三大特性：`有序性，可见性，原子性`。而synchronized作用于存储在内存中的变量或代码块上，它除了拥有内存模型的三大特性外，还有`可重入性`，谈到可重入性首先要知道synchronized是如何实现加锁和解锁的。

##### 加锁与解锁

我们将**class文件反编译**可以得到：

![image-20210328101501442](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210328101501442.png)

对应来看，加锁和解锁显然是通过**monitorenter和monitorexit**实现的。也就是说获取对象的锁，实质上就是获取对象的monitor，即**监视器**，那如何来标识线程拥有该监视器呢？synchronized在锁对象时会获得一个**计数器**，用来**标识线程获得锁的次数**，线程每重入一次锁，计数器就会加1，对应的线程每调用monitorexit退出一次锁，计数器就会减1.如果计数器减为0，则说明锁已被释放。

这是线程获得锁的情况，但假如线程阻塞了呢——

![img](https://pdai.tech/_images/thread/java-thread-x-key-schronized-2.png)

显然，如果获取失败，线程就会进入**同步队列**，状态变更为**阻塞态**，直到锁被释放以后，同步队列中的线程才有机会获得锁。

##### 可重入性原理

现在来分析可重入性，就变得非常友好了。线程在重入时，`会首先判断计数器的值`，如果计数器不为0，也就是线程拥有锁，则直接进入，不需要反复获取锁。

发生死锁的必要条件之一即为`循环等待`，而可重入性打破了这一原因，可见`可重入性能够有效防止死锁的出现`。

##### 同步方法的底层实现

同步方法区别于代码块，它会首先判断标志位**ACC_SYNCHRONIZED**,如果存在该标志位，才会去隐式的调用monitorenter与monitorexit指令。即最终都归于monitor对象。

我们所熟识的锁升级，无非是调用了**ObjectMonitor对象不同的实现**，如果失败了，再调用更高级的实现即可。

##### 重量锁

在看ObjectMonitor的源码时，我们会发现内核函数的存在，它们分别对应**park()和unpark()**,而synchronized是JVM管理的，JVM是应用程序，处于用户态，而内核函数在内核态，`切换的过程耗费资源较低，因此得名重量锁`。

再来简单回顾下内核态与用户态的切换——

1. 用户态将数据存储在`寄存器`中，并保存堆栈信息；
2. 通过`系统调用`的方式，即通过`外中断`进入内核态；
3. `CPU取指执行`，即跳转到指定的内存位置处执行指令；
4. `读取寄存器中的参数`，维护内核栈的堆栈信息；
5. CPU`重置操作系统为内核态`。

#### JVM中锁的优化

既然用户态，内核态的切换是低效的，那我们不如**只在用户态做文章**，官方对锁的升级即是通过加函数调用，而非系统调用来实现的。首先概括性的看下图——

<img src="https://pic2.zhimg.com/v2-79edfb4b2316d76ac653732fbdb72809_r.jpg" alt="preview" style="zoom:200%;" />

这张图着实有点烧脑，我们来看个简单的：

![image-20210328105848069](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210328105848069.png)

这就是锁升级的方向咯。显然这个过程是不可逆的，我们来拆解它的每一步实现——

- **偏向锁**

  之前我们提到了，对象头是由Mark_Word和Klass_point组成，锁争夺也就是对monitor对象的争夺，`一旦线程持有该对象，标志位修改为1，进入偏向模式`。并将这个线程的ID记录在对象的Mark_Word中。

  这个过程是使用`CAS乐观锁`来实现的，如果CAS失败，即锁对象获取失败。

  偏向锁在`jdk1.6之后是默认开启`的，1.5中关闭。

  ![preview](https://pic1.zhimg.com/v2-1c56b94b3f92c3aa3884f739a5e4b42c_r.jpg)

  如果多个线程竞争偏向锁，那CAS乐观锁显然不那么实用了，就需要用到轻量级锁——

- **轻量级锁**

  轻量级锁的实现仍跟Mark_Word相关。如果对象是无锁的，JVM就会在当前线程的栈帧中创建一个叫做`锁记录(Lock Record)的空间`，用来存储锁对象`Mark_Word的拷贝`。最后将所记录的`owner字段指向当前对象`。

  JVM接下来会利用CAS尝试将对象原本的`Mark_Word更新为Lock_Record的指针`。如果更新成功，改变锁的标志位，表明对象的锁已被获取，然后执行相关同步操作。

  如果失败了，说明`锁可能被其他线程持有`，此时就需要继续升级锁，修改锁的状态，线程进入阻塞态。而还有一种可能，即当前`线程本身持有锁`，就需要判断当前对象的Mark_Wors是否指向当前线程的栈帧。

  ![preview](https://pic3.zhimg.com/v2-a932f172d1e9ac967c213429054d8522_r.jpg)

- **自旋锁**

  上面提到如果修改不成功，就需要升级为自旋锁，自旋锁区别于重量锁，它如果没有竞争到锁对象，`就会不断循环，表现为CPU空转，而不是阻塞`。

  空转的次数最多为10次，如果超过10次，就会升级为重量锁。

------

#### 锁的优缺点对比

![image-20210328112941479](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210328112941479.png)

------

### 3.lock

剧情来到了互斥并发的半壁江山——lock。

lock本身是一个`接口`，它提供了`更灵活的锁定操作`，即`手动的加锁与释放锁`。完全`不同的属性`，并且支持多个关联的对象`Condition`。

特别地，它的实现类`ReadWriteLock`支持对共享资源的读锁。也可以使用`tryLock方法非阻塞`地获得锁。

下面我们通过具体的实现类来看lock的实现机制——

#### ReentrantLock

![image-20210328124145535](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210328124145535.png)

ReentrantLock顾名思义，即`可重入的锁`。它的属性包含同步类的对象，该对象继承自AQS，即使用`双端队列实现的同步队列`。

在来看下它的属性与方法——
![image-20210328124815007](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210328124815007.png)

嗯，提供了很多便捷，灵活的操作锁的方式，这里不再一一赘述，因为相比Lock的API，显然`JUC包下的API更香`~

### 4.synchronized VS ReentrantLock

- **重入**

  `synchronized可重入`，因为加锁和解锁可以自动进程，不必担心最后释放锁。`ReentrantLock同样可以重入`，它的加锁和释放锁需要手动完成。

- **实现**

  `synchronied是JVM实现的`，而`ReentrantLock是JDK实现的`，也就是最终归于操作系统实现。

- **性能**

  jdk1.6以后对synchronied做了很多优化，。如今它们的性能平分秋色。

- **功能**

  ReentrantLock锁的粒度和灵活度，都明显优于synchronized.

  ReentrantLock从它内部类可以看出，它有公平与非公平的同步方式之分。而`synchronized是非公平(不按时间顺序获取锁)的锁。`

  ReentrantLock可以非阻塞的获取锁。且提供中断等待锁的线程机制。

- **可见性**

  我们知道，`synchronized的可见性是基于java内存模型的happens-before原则`，即在其他线程读取主内存的值前，主内存一定先被刷新。

  而ReentrantLock的可见性是如何保障的呢？它的同步机制是基于AQS框架的，而`AQS框架的锁机制核心在于读写state变量`的值：

  ![image-20210328152434820](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210328152434820.png)

这里简要插播一下：volatile是如何保证可见性的：

> volatile基于JMM(java内存模型)的内存语义：
>
> 1. 当`写一个volatile变量`时，JMM会将线程本地内存中的变量刷新到主内存中；
> 2. 当`读一个volatile变量`时，JMM会将线程对应的本地内存置为无效，然后从主内存中读取变量。

那state变量既然由volatile修饰，那自然就可以保证可见性咯，当然，保证原子性是不可能的~

## 多线程——黄金时代

### 1.线程池

#### 什么是线程池

线程池的基本思想是`对象池`，当程序启动的时候，就会在内存中开辟出一块空间，来放置初始化好的线程。当需要使用线程时，就使用`线程调度器`从线程池中取出一个线程，当线程使用完后，再将它归还到线程池中。

#### 使用线程池的好处

- `方便管理线程`：比如延时执行，定时循环；
- `控制线程并发数量`：如果线程数量不加控制，会大致线程占用内存过多，从而四死机；
- `减少创建，销毁开销`：每个线程可以被重复利用，可以执行多个任务

#### 线程池的主要组件

![img](https://user-gold-cdn.xitu.io/2018/7/8/16477f7912b4552a?imageslim)

1. `线程池管理器ThreadPool`：用于创建和管理线程池；
2. `任务接口Task`：每个任务必须实现的接口，以供工作线程调度任务的执行，它主要规定了任务的入口，执行完的收尾工作，任务的执行状态；
3. `工作线程workThread`：线程池中的线程，没有任务时处于等待状态；
4. `任务队列taskQueue`：用于存放没有处理的任务，起一种`缓冲的作用`。

#### ThreadPoolExecutor类解析

我们可以通过ThreadPoolExecutor来创建一个线程——

![image-20210328164241430](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210328164241430.png)

再来深入地看下这些参数的意义：

1. `corePoolSize`：即线程池的基本大小，任务队列每提交一个任务，线程池就会创建一个线程，当提交的任务数大小线程池的基本大小时，就不再创建线程；
2. `maxinumPoolSize`：即线程池允许创建的最大线程数；
3. `KeepAliveTime`：即线程池的工作线程空闲了以后，保持存活的时间；
4. `TImeUnit`：即线程活动保持时间的单位；
5. `workQueue`：用来保存等待执行的阻塞队列；
6. `threadFactory`：线程工厂，在debug和定位问题时非常有帮助；
7. `RejectedExecutionHandler`：拒绝策略，下文会详细描述。

##### 向线程池提交任务

- `execute()`：无返回值，无法判断任务是否被执行成功；
- `submit()`：返回一个future的泛型对象，通过get方法可以获得返回值。

##### 线程池的关闭

- `shutdown`：将线程池的状态设置为`SHUTDOWN状态`，然后中断所有没有执行任务的线程；
- `shutdownNow`：遍历工作线程，逐个调用线程的`interrupt方法`。

##### 线程池的执行流程

![img](https://user-gold-cdn.xitu.io/2018/7/8/16477fc44a03d90f?imageslim)

##### 四种拒绝策略

1. `AbortPolicy`：丢弃任务并抛出RejectedExecutionException异常

   ![	](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210328171249081.png)

2. `DiscardPolicy`：丢弃任务但不抛出异常

   ![image-20210328171502262](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210328171502262.png)

3. `DisCardOldSetPolicy`：丢弃队列最前面的任务，然后提交新来的任务

   ![image-20210328171523558](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210328171523558.png)

4. `CallerRunPolicy`：由调用线程处理该任务

   ![image-20210328171435296](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210328171435296.png)

#### 四种线程池

- `CachedThreadPool()`:可缓冲线程池
- `FIxedThreadPool()`:定长线程池
- `ScheduledThreadPool()`:定时线程池
- `SingleThreadExecutor()`:单线程化的线程池

#### 为什么不推荐使用JUC的线程池

1. newFixedThreadPool和newSingleThreadExecutor

   创建的`任务等待队列容量过大，耗费非常多的内存`~

   ![image-20210329201950011](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210329201950011.png)

   ![image-20210329202057662](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210329202057662.png)

2.newScheduledThreadPool和newCachedThreadPool

​	会`创建非常多的线程`，CPU压力大~

​	![image-20210329202312593](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210329202312593.png)

#### 线程池的参数设置

参数的设置和系统的负载有着直接的关系，系统负载有这样的一些参数：

- `tasks`：每秒需要处理的任务数
- `threadtasks`：每个线程每秒可以处理的任务数
- `responsetime`：系统允许任务的最大响应时间。

下面将用这些系统负载的参数来衡量线程池的参数——

##### corePoolSize

假设系统每秒任务数为100 ~ 1000，每个线程每钞可处理10个任务，则需要100 / 10至1000 / 10，即10 ~ 100个线程。那么corePoolSize应该设置为大于10，具体数字最好根据`8020原则`，因为系统每秒任务数为100 ~ 1000，`即80%情况下系统每秒任务数小于1000 * 20% = 200`，则corePoolSize可设置为`200 / 10 = 20`。

##### queueCapacity

任务队列的长度可以设置为：`所有核心线程每秒处理任务数 * 每个任务响应时间 = 每秒任务总响应时间`

##### maxPoolSize

当系统负载达到最大值时，核心线程数已无法按时处理完所有任务，这时就需要增加线程，增加的线程数取决于任务队列容量和需求。

千万不能直接创建任务队列，不然容量过大，占用内存过多，对系统压力大。

![image-20210329204208430](C:\Users\zyz\AppData\Roaming\Typora\typora-user-images\image-20210329204208430.png)

##### keepAliveTime

当`负载降低时，可以减少线程的数量`，当线程的空闲时间超过keepAliveTime，会自动释放线程。

##### allowCoreThreadTimeout

默认情况下核心级线程不会退出，可通过该参数设置为true,使其退出。

一般经验：线程池的大小应该这样设置：

1. 如果是`CPU密集型应用`，线程池大小设置为`N+1`,N指CPU的个数；
2. 如果是`IO密集型应用`，则线程池大小设置为`2N+1`;

#### 线程的五种状态

![img](https://user-gold-cdn.xitu.io/2020/3/20/170f73cc0dad76a5?imageslim)

#### 什么场景下怎么设置线程池

- `高并发、任务执行时间短的业务`

线程池线程数可以设置为CPU核数+1，**减少线程上下文的切换**

- `并发不高、任务执行时间长的业务`

这个需要判断执行时间是耗在哪个地方

1. 假如是业务时间长集中在`IO操作上`，也就是IO密集型的任务，因为IO操作并不占用CPU，所以不要让所有的CPU闲下来，可以适当加大线程池中的线程数目（**2 * CPU核数**），让CPU处理更多的业务。
2. 假如是业务时间长集中在计算操作上，也就是`CPU密集型任务`，和（1）**CPU核数+1** 一样吧，线程池中的线程数设置得少一些，减少线程上下文的切换

- `并发高、业务执行时间长的业务`

解决这种类型任务的关键不在于线程池而在于整体架构的设计

### 2.JUC

#### AQS框架

我们知道JUC是一个**定义并发操作并安全处理并发**的框架。而要了解JUC包的全貌，我们首先需要定义处理`并发的组件`，也就是锁和同步器。这种功能就是由AQS所提供的。

AQS的全称是`AbstractQueuedSynchronizer`，即抽象队列同步器。它是java并发用来构建锁和同步器的基础框架。它包含`先进先出的双端队列，状态量，线程持有者的引用`。

![img](https://upload-images.jianshu.io/upload_images/24459569-f4323d6ace89b32d.png)

这里的state是由volatile修饰的，即它满足线程间的可见性，另外，它记录了`锁的持有者`：

![image-20210329223019662](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210329223019662.png)

当线程1想要获取monitor对象时，它的行为是这样的：

![image-20210329224019027](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210329224019027.png)

------

#### 工具类——同步器

累了，不再文字描述了，把思维导图贴上来：

![三大工具类-同步器](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/三大工具类-同步器.png)

#### 其他组件

![其他组件](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/其他组件.png)

#### ThreadLocal

[浅谈ThreadLocal](https://zhuanlan.zhihu.com/p/60375306)

## 多线程——加餐

### 1.关于多线程可行的实践

- 给线程起个`有意义的名字`，这样可以方便找 Bug。
- 缩小同步范围，从⽽而减少锁争用。例如对于 synchronized，应该`尽量使用同步块⽽而不是同步方法`。
- 多用同步工具少用 wait() 和 notify()。首先`，CountDownLatch, CyclicBarrier, Semaphore 和Exchanger 这些同步类简化了了编码操作，⽽用 wait() 和 notify() 很难实现复杂控制流`；其次，这些同步类是由最好的企业编写和维护，在后续的 JDK 中还会不不断优化和完善。
- 使用`BlockingQueue 实现生产者消费者问题`。多用并发集合少用同步集合，例如应该`使用 ConcurrentHashMap ⽽而不是 Hashtable`。
- 使用`本地变量`和`不可变类`来保证线程安全。使用线程池⽽不是直接创建线程，这是因为创建线程代价很高，线程池可以有效地利用有限的线程来启动任务。

------

> 山高路远，静水深流.