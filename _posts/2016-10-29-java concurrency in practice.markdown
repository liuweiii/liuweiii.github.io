---
layout: post
title:  "java concurrency in practice"
date:   2016-10-29 21:20:00 +0800
categories: java
tags:
- jsr 166
- learn
- java
---

## 1.线程安全性##

### 1.1.线程安全性### 

当多个线程访问某个类时，这个类始终都能表现出正确的行为，那么就称这个类是线程安全的。

*当多个线程访问某个类时，不管运行时环境采用何种调度方式或者这些线程将如何交替运行，并且在主调代码中不需要任何额外的同步或协同，这个类都能表现出正确的行为，那么就称这个类是线程安全的。*

无状态对象一定是线程安全的。

### 1.2.原子性###

i++和++i都不是原子操作，实际上包含了3个独立操作：读取值，将值加1，将结果写入。

#### 1.2.1.竞态条件####

在并发编程中，由于不恰当的执行时序而出现不正确的结果，就成为竞态条件。

当某个计算的正确性取决于多个线程交替执行时序时，就会发生竞态条件，换句话说，就是正确的结果要取决于运气。最常见的竞态条件类型就是“先检查后执行”操作，即通过一个可能失效的观测结果来决定下一步的动作。

*假定两个操作A和B，如果从执行A的线程来看，另一个线程执行B时，要么将B全部执行完，要么完全不执行B，那么A和B对彼此来说就是原子的。*

#### 1.2.2.复合操作####

将“先检查后执行”和“读取-修改-写入”等操作称为复合操作：包含了一组必须以原子方式执行的操作以确保线程安全性。

### 1.3.加锁机制###

*要保持状态的一致性，就需要在单个原子操作中更新所有相关的状态变量*

#### 1.3.1.内置锁####

{% highlight java linenos %}
synchronized (lock){
  //
}
{% endhighlight %}

Java的内置锁相当于一种互斥锁，意味着最多只有一个线程能持有这种锁。

#### 1.3.2.重入####

当某个线程试图获得一个已经由自己持有的锁，这个请求会成功。重入的一种实现方法是，为每个锁关联一个获取计数值和一个所有者线程。

### 1.4. 用锁来保护状态###

*对于可能被多个线程同时访问的可变状态变量，在访问它时都需要持有同一个锁，在这种情况下，我们称状态变量是由这个锁保护的。*

当获取与对象关联的锁时，并不能阻止其他线程访问该对象，只能阻止其他线程获得同一个锁。之所以每个对象都有一个内置锁，只是为了免去显式地创建锁对象。

一种常见的加锁约定是，将所有可变状态都封装在对象内部，并通过对象的内置锁对所有访问可变状态的代码路径进行同步，使得在该对象上不会发生并发访问。

并非所有数据都需要锁的保护，只有被多个线程同时访问的可变数据才需要通过锁来保护。

*对于每个包含多个变量的不变性条件，其中涉及的所有变量都需要由同一个锁来保护*。

### 1.5.活跃性与性能###

*在简单性与性能之间存在着相互制约因素，当实现某个同步策略时，一定不要盲目地为了性能而牺牲简单性（可能会破坏安全性）。*

*当执行时间较长的计算或者可能无法快速完成的操作时（如网络I/O或控制台I/O），一定不要持有锁。*

## 2.对象的共享##

### 2.1.可见性###

加锁的含义不仅仅局限于互斥行为，还包括内存可见性。为了确保所有线程都能看到共享变量的最新值，所有执行读操作或写操作的线程都必须在同一个锁上同步。

加锁机制既可以确保可见性又可以确保原子性，而volatile变量只能确保可见性。

### 2.2.发布与逸出###

**发布**： 讲一个指向该对象的引用保存到其他代码可以访问的地方，或者在某个非私有的方法中返回该引用，或者将引用传递到其他类的方法中。

**逸出**：发布某个不应该发布的对象时，称为逸出。

很多情况下，要确保对象及其内部状态不被发布；而某些情况下，又需要发布某个对象，但如果在发布时要确保线程安全性，则可能需要同步。发布内部状态可能会破坏封装性，使得程序难以维持不变性条件。例如，在对象构造完成之前就发布该对象，会破坏线程安全性。

发布的几种方式：

1. 将对象引用保存到一个公有静态变量中，以便任何类和线程都能看见该对象。

2. 从非私有方法中返回一个引用，同样会发布返回的对象。

3.  发布一个内部的类实例

*不要在构造过程中使this引用逸出。*

在构造过程中使this引用逸出的一个常见错误是，在构造函数中启动一个线程。在对象的构造函数中创建一个线程时，this引用都会被新创建的线程共享。在对象尚未完全构造之前，新的线程就可以看见它。在构造函数中创建线程并没有错，但最好不要立即启动它，而是通过一个start或initialize方法来启动。

{% highlight java linenos %}
public class Test {

    private boolean isIt;

    public Test() throws InterruptedException {
        new Thread(new Runnable() {
            public void run() {
                System.out.println(isIt);
            }
        }).start();
        Thread.sleep(2000L);
        isIt = true;
    }

    public static void main(String[] args) throws InterruptedException {
        Test test = new Test();
    }
}
{% endhighlight %}
打印的结果是false，这个例子就是隐式的this对象引用逸出，还没有实例化完成时，其他线程就已经要用到对象中的属性

*stackoverflow上的一个例子(如果有多个实例在多个线程中被初始化，某些线程用的this可能是未被完全初始化的)：*
{% highlight java linenos %}
public class Test
{
    private static Test lastCreatedInstance;

    public Test()
    {
        lastCreatedInstance = this;
    }
}
{% endhighlight%}

书（《Java并发编程实战》）上的会this逸出例子是：
{% highlight java linenos %}
public class ThisEscape {
  public ThisEscape(EventSource source) {
    source.registerListener(new EventListener() {
      public void onEvent(Event e) {
        doSomething(e);
      }
    });
}
{% endhighlight %}
给出的正解是：
{% highlight java linenos %}
public class SafeListener {
  private final EventListener listener;

  private SafeListener() {
    listener = new EventListener() {
      public void onEvent(Event e) {
        doSomething(e);
      }
    };
  }

  public static SafeListener newInstance(EventSource source) {
    SafeListener safe = new SafeListener();
    source.registerListener(safe.listener);
    return safe;
  }
}
{% endhighlight %}
当构造好了SafeListener对象之后，我们才启动了监听线程，也就确保了SafeListener对象是构造完成之后在使用的SafeListener对象。

### 2.3. 线程封闭###

即对象只能由单个线程访问。

当某个对象封闭在一个线程中时，这种用法将自动实现线程安全性。

线程封闭的几种方法：
**1. Ad-hoc线程封闭** ？？？ 维护线程封闭性的职责完全由程序实现来承担，Ad-hoc线程封闭性是非常脆弱的，因为没有任何一种语言特性，例如可见性修饰符或局部变量，能将对象封闭到目标线程上。、

**2. 栈封闭**  只能通过局部变量访问对象。

**3. ThreadLocal类**  维持线程封闭性的一种更规范方法是使用ThreadLocal，这个类能使线程中的某个值与保存值的对象关联起来。ThreadLocal提供了get与set等接口或方法，这些方法为每个使用该变量的线程都存有一份独立的副本，因此get总是返回由当前执行线程调用set时设置的最新值。

ThreadLocal的作用是提供线程内的局部变量，这种变量在线程的生命周期内起作用，减少同一个线程内多个函数或组件之间一些公共变量的传递复杂度。

提供线程内部的局部变量，在本现场内随时随地可取，隔离其他线程。


### 2.4. 不变性###

*不可变对象一定是线程安全的。*

final类型的域是不能修改的（但如果final域所引用的对象是可变的，这些被引用的对象是可以修改的。另，通过反射可以修改final域），此外，final域能确保初始化过程的安全性，从而可以不受限制地访问不可变对象，并在共享这些对象时无须同步。

*修改final域，static 的final不能修改*
{% highlight java linenos %}
public class MyFinal {
    private final int a = 1;
    private static final int b = 1;
    private static void out(String msg){
        System.out.println(msg);
    }
    public static void Test() throws NoSuchFieldException, IllegalAccessException {
        MyFinal f = new MyFinal();
        out("`final` field can be modified.");
        out("before: a="+f.a);

        Field field = f.getClass().getDeclaredField("a");
        field.setAccessible(true);
        field.set(f,2);
        out("after: a="+f.a);

        out("`static final` filed can't be modified.");
        out("before: b="+f.b);

        Field field2 = f.getClass().getDeclaredField("b");
        field2.setAccessible(true);
        field2.set(f, 2);
        out("after: b="+f.b);
    }
    public static void main(String[] args) throws  NoSuchFieldException,IllegalAccessException{
        Test();
    }
}
{% endhighlight %}

{% highlight java linenos %}
·`final` field can be modified.
before: a=1
after: a=1
`static final` filed can't be modified.
before: b=1
Exception in thread "main" java.lang.IllegalAccessException: Can not set static final int field org.liuwei.learn.finalField.MyFinal.b to java.lang.Integer
	at sun.reflect.UnsafeFieldAccessorImpl.throwFinalFieldIllegalAccessException(UnsafeFieldAccessorImpl.java:76)
	at sun.reflect.UnsafeFieldAccessorImpl.throwFinalFieldIllegalAccessException(UnsafeFieldAccessorImpl.java:80)
	at sun.reflect.UnsafeQualifiedStaticIntegerFieldAccessorImpl.set(UnsafeQualifiedStaticIntegerFieldAccessorImpl.java:77)
	at java.lang.reflect.Field.set(Field.java:764)
	at org.liuwei.learn.finalField.MyFinal.Test(MyFinal.java:30)
	at org.liuwei.learn.finalField.MyFinal.main(MyFinal.java:34)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)

Process finished with exit code 1
{% endhighlight %}

*正如“除非需要更高的可见性，否则应将所有域声明为私有域“是一个良好的编程习惯，”除非需要某个域是可变的，否则应将其声明为final域“，也是一个良好的编程习惯。*

### 2.5. 安全发布###

不正确的发布导致正确的对象被破坏。

#### 2.5.1. 不可变对象与初始化安全性####

即使在发布不可变对象引用时没有使用同步，也仍然可以安全地访问该对象。为了维持这种初始化安全性的保证，必须满足不可变性的所有需求：状态不可修改，所有域都是final类型，以及正确的构造过程。

#### 2.5.2. 安全发布的常用模式####

要安全地发布一个对象，对象的引用以及对象的状态必须同时对其他线程可见。一个正确构造的对象可以通过以下方式来安全地发布：

- 在静态初始化函数中初始化一个对象引用。

- 将对象的引用保存到volatile类型的域或者AtomicReferance对象中。

- 将对象的引用保存到某个正确构造对象的final类型域中。

- 将对象的引用保存到一个由锁保护的域中。

线程A将对象X放入一个线程安全的容器，随后线程B读取这个对象，那么可以确保B看到A设置的X状态，即便在这段读/写X的应用程序代码中没有包含显示的同步。

通常，要发布一个静态构造的对象，最简单和最安全的方式是使用静态初始化器：

{% highlight java linenos %}
public static Holder holder = new Holder(42);
{% endhighlight %}

静态初始化器由JVM在类初始化阶段执行。由于在JVM内部存在者同步机制，因此通过这种方式初始化的任何对象都可以被安全地发布。

#### 2.5.3. 事实不可变对象####

如果对象从技术上来看是可变的，但其状态在发布后不会再改变，那么把这种对象称为“事实不可变对象”。

#### 2.5.4. 可变对象####

对象的发布需求取决于它的可变性：

- 不可变对象可以通过任意机制来发布。

- 事实不可变对象必须通过安全方式来发布。

- 可变对象必须通过安全方式来发布，而且必须是线程安全的或者由某个锁保护起来。

#### 2.5.5. 安全地共享对象####

在并发程序中使用和共享对象时，可以使用一些实用的策略，包括：

**线程封闭**线程封闭的对象只能由一个线程拥有，对象被封闭在该线程中，并且只能由这个线程修改。

**只读共享**在没有额外同步的情况下，共享的只读对象可以由多个线程并发访问，但任何线程都不能修改它。共享的只读对象包括不可变对象和事实不可变对象。

**线程安全共享**线程安全的对象在其内部实现同步，因此多个线程可以通过对象的公有接口来访问而不需要进一步的同步。

**保护对象**被包含的对象只能通过持有特定的锁来访问。保护对象包括封装在其他线程安全对象中的对象，以及已发布的并且由某个特定锁保护的对象。

## 3. 对象的组合##

### 3.1. 设计线程安全的类###

在设计线程安全类的过程中，需要包含一下三个基本要素：

- 找出构成对象状态的所有变量。

- 找出约束状态变量的不变性条件。

- 建立对象状态的并发访问管理策略。

许多情况下，所有权与封装性总是相互关联的：对象封装它拥有的状态，反之也成立，即对它封装的状态拥有所有权。然而，如果发布了某个可变对象的引用，那么就不再拥有独占的控制权，最多是“共享控制权”。

对于从构造函数或从方法中传递进来的对象，类通常并不拥有这些对象，除非这些方法是被专门设计为转移传递进来的对象的所有权（如，同步容器封装器的工厂方法）。

容器类通常表现出一种“所有权分离”的形式，其中容器类拥有其自身的状态，而客户代码则拥有容器中各个对象的状态。

### 3.2. 实例封闭###

将数据封装在对象内部，可以将数据的访问限制在对象的方法上，从而更容易确保线程在访问数据时总能持有正确的锁。

*封闭机制更易于构造线程安全的类，因为当封闭类的状态时，在分析类的线程安全性时就无须检查整个程序。*

遵循Java监视器模式的对象会把对象的所有可变状态都封装起来，并由对象自己的内置锁来保护。许多类中都使用了Java监视器模式，如Vector和Hashtable。

Java监视器模式仅仅是一种编写代码的约定，对于任何一种锁对象，只要自始至终都使用该锁对象，都可以用来保护对象的状态。

**使用私有锁来保护状态**
{% highlight java linenos %}
public class PrivateLock{
  private final Object myLock = new Object();
  Widget widget;
  void someMethod(){
    synchronized(myLock){
      // visit widget;
    }
  }
}
{% endhighlight %}

使用私有锁对象而不是对象的内置锁（或任何其他可通过公有方式访问的锁）。私有的锁对象可以将锁封装起来，使客户代码无法得到锁，客户代码可以通过公有方法访问锁。

在某些情况下，通过多个线程安全类组合而成的类是线程安全的，而在某些情况下，这仅仅是一个好的开端。

*【m】如果一个对象需要被多个线程安全地使用，【简单来做】：首先不能让该对象的字段逸出（不要有非final的public字段，且如果是final的public字段也可能逸出，如public ArrayList<Object>，这个ArrayList的Object就是可变的；不要通过方法把非final字段的引用扔出去，扔出去的final字段对象中不要包含可变的字段），然后把对象的每个方法都synchronized。这样保证了线程安全，但是并发大的时候性能降低。【好的方法（只是安全了，不一定比前一个好）】：不实用synchronized，但保证所有字段都是final的，且是安全发布的。*

*【m】类的多个状态各自都是线程安全的，但经过组合得到的这个类却不一定是线程安全的，因为这些状态间彼此可能不是独立的，这时可以通过加锁机制来维护不变性条件以确保其线程安全性。同时还得避免发布这些状态，防止客户代码破坏其不变性条件。*

*如果一个状态变量是线程安全的，并且没有任何不变性条件来约束它的值，在变量的操作上也不存在任何不允许的状态转化，那么就可以安全地发布这个变量。*

## 4. 基础构建模块##

### 4.1. 同步容器类###

同步容器类包括Vector和Hashtable，二者是早期JDK的一部分，此外还包括在JDK1.2中添加的一些功能相似的类，这些同步的封装器类是由Collections.synchronizedXXX等工厂方法创建的。这些类实现线程安全的方法是：将它们的状态封装起来，并对每个公有方法都进行同步，使得每次只有一个线程能访问容器的状态。

Vector 使用了synchronized线程安全，但是并发性能差。当它在迭代过程中被修改时，会抛出ConcurrentModificationException异常。这种“及时失败（fail-fast）”的迭代器不是一种完备的处理机制，而只是“善意地”捕获并发错误，因此只能作为并发问题的预警指示器。

*正如封装对象的状态有助于维持不变性条件一样，封装对象的同步机制同样有助于确保实施同步策略。*

Java5.0提供了多种并发容器类来改进同步容器的性能。同步容器将所有对象状态的访问都串行化，以实现它们的线程安全性。

#### 4.1.2. 并发容器####

**ConcurrentHashMap** 使用一种粒度更细的加锁机制（分段锁）来实现更大程度的共享，实现在并发访问环境下更高的吞吐量，而在单线程环境中只损失非常小的性能。不会抛出ConcurrentModificationException。是弱一致性的。如，由于size返回的结果在计算时可能已经过期了，它实际上只是一个估计值，因此允许size返回一个近似值而不是一个精确值。虽然这看上去有些令人不安，但事实上size和isEmpty这样的方法在并发环境下的用处很小，因为它们的返回值总在不断变化。因此这些操作的需求被弱化了，以换取对其他更重要操作的性能优化，包括get、put、containsKey和remove等。

*大多数情况下，用ConcurrentHashMap来代替同步Map能进一步提高代码的可伸缩性，只有需要加锁Map以进行独占访问时，才应该放弃使用ConcurrentHashMap*。

**CopyOnWriteArrayList**用于替代同步List，在每次修改时，都会创建并重新发布一个新的容器副本，从而实现可变性。多个线程可以对这个容器进行迭代，而不会彼此干扰或者与修改容器的线程相互干扰。“写入时复制”容器返回的迭代器不会抛出ConcurrentModificationException，并且返回的元素与迭代器创建时的元素完全一致，不必考虑之后修改操作所带来的影响。

*仅当迭代操作远远多余修改操作时，才应该使用CopyOnWriteArrayList。例如事件通知系统，在大多数情况下，注册和注销事件监听的操作远少于接收事件通知的操作。*

### 4.2. 阻塞队列和生产者-消费者模式###

*在构建高可靠的应用程序时，有界队列是一种强大的资源管理工具：它们能抑制并防止产生过多的工作项，使应用程序在负荷过载的情况下变得更加健壮。*

在类库中包含了BlockingQueue的多种实现，其中，LinkedBlockingQueue和ArrayBlockingQueue是FIFO队列，二者分别与LinkedList和ArrayList类似，但比同步List拥有更好的并发性能。

### 4.3. 同步工具类###

同步工具类包括：阻塞队列、信号量（Semaphore）、栅栏（Barrier）以及闭锁（Latch）。

所有同步工具类都包含一些特定的结构化属性：它们封装了一些状态，这些状态将决定执行同步工具类的线程是继续执行还是等待，此外还提供了一些方法对状态进行操作，以及另一些方法用于高效地等待同步工具类进入到预期状态。

#### 4.3.1. 闭锁####

闭锁的作用相当于一扇门，在闭锁到达结束状态前，这扇门是关闭的，没有任何线程能通过，当到达结束状态时，这扇门会打开并允许所有线程通过。例如，在多玩家游戏中，所有玩家都进入了再开始。

CountDownLatch是一种灵活的闭锁实现。它的状态包括一个计数器，该计数器被初始化为一个正数，表示需要等待的事件数量。countDown方法递减计数器，表示有一个事件已经发生了，而await方法等待计数器达到零，表示所有需要等待的事件都已经发生。如果非零，await会一直阻塞，或者等待中的线程中断，或者等待超时。

#### 4.3.2. FutureTask####

FutureTask也可以用作闭锁。（FutureTask实现了Future语义，表示一种抽象的可生成结果的计算）。FutureTask表示的计算是通过Callable来实现的，想到与一种可生成结果的Runnable。可以处于以下3种状态：*等待运行*、*正在运行*、*运行完成*。当FutureTask进入完成状态后，会永远停止在这个状态上。

FutureTask在Executor框架中表示异步任务，此外还可以表示一些时间较长的计算，这些计算可以在使用计算结果之前启动。

*所有并发问题都可以归结为如何协调对并发状态的访问。可变状态越少，就越容易确保线程安全性。*

#### 4.3.3. 信号量####

计数信号量用来控制同时访问某个特定资源的操作数量，或者同时执行某个制定操作的数量，还可以用来实现某种资源池（如数据库连接池），或对容器施加边界。

Semaphore中管理着一组虚拟的许可，许可的初始数量可通过构造函数来制定。执行操作时先获得许可（acquire），使用后释放（release）。acquire将阻塞直到有许可（或被中断，或操作超时）。release将返回一个许可给信号量。

Semaphore不会将许可与线程关联，因此在一个线程中获得的许可可以在另一个线程中释放。

信号量可以简化为二值信号量，即初始值为1的Semaphore。二值信号量可以作为互斥体（mutex），并具备不可重入的加锁语义。

#### 4.3.4. 栅栏####

栅栏（Barrier）类似于闭锁，能阻塞一组线程直到某个事件发生。栅栏与闭锁的区别在于，所有线程必须同时到达栅栏位置，才能继续执行。

闭锁用于等待事件，而栅栏用于等待其他线程。

一般使用CyclicBarrier。

## 5. 任务执行##

### 5.1. Executor框架###

在Java类库中，任务执行的主要抽象不是Thread，而是Executor。
{% highlight java linenos %}
public interface Executor{
  void execute(Runnable command);
}
{% endhighlight %}

Executor的用法：
{% highlight java linenos %}
...
private static final Executor exec = Executors.newFixedThreadPool(10);
//newFixedThreadPool 返回的是ThreadPoolExecutor，
//其继承自ExecutorService，可以强转成ExecutorService，
//然后使用exec.submit来执行Callable的任务，
//然后使用exec.shutdown来关闭线程池，不然JVM不会退出。
...
Runnable task = new Runnable ...
...
exec.execute(task);
...
{% endhighlight %}
可以很容易地修改程序的行为，只需要编写Executor的子类，如下：
{% highlight java linenos %}
public class NewExecutor implements Executor{
  public void execute(Runnable r){
    new Thread(r).start();//这里可以修改行为，
    //如写为 r.run();    
  }
}
{% endhighlight %}

*每当看到下面这种形式的代码时：*
{% highlight java linenos %}
new Thread(runnable).start();
{% endhighlight %}
*并且希望获得一种更灵活的执行策略时，请考虑使用Executor来代替Thread。*

### 5.2. 线程池###

类库提供了一个灵活的线程池以及一些有用的默认配置，可以通过调用Executors中的静态工厂方法之一来创建一个线程池：

**newFixedThreadPool**创建一个固定长度的线程池。每提交一个任务时就创建一个线程，直到达到最大数量；如果某个线程以外挂了，会补充一个新的线程。

**newCachedThreadPool**创建一个可缓存的线程池，如果线程池规模超过了处理需求，将回收空闲的线程，而需求增加时，可以添加新的线程，线程池规模不存在限制。

### 5.3. Executor的生命周期###

Executor的实现通常会用来创建线程执行任务，但JVM只有在所有（非守护）线程全部终止后才会退出，ExecutorService需要使用shutdown来关闭，不然JVM不会退出。

## 6.取消与关闭##

*在Java的API或语言规范中，并没有将中断与任何取消语义关联起来，但实际上，如果在取消之外的其他操作中使用中断，都是不合适的。*

每个线程都有一个boolean类型的中断状态。当线程中断时，这个状态将被设置为true。

{% highlight java linenos %}
public class Thread{
  public void interrupt(){...}//中断目标线程
  public boolean isInterrupted(){...}//查询中断状态
  public static boolean interrupted(){...}//清除中断状态
}
{% endhighlight %}

Thread.sleep、Object.wait等，都会检查线程何时中断，在发现中断时提前返回。它们在响应中断时执行的操作包括：清除中断状态，抛出InterruptedException。

*调用interrupt并不意味着立即停止目标线程正则进行的工作，而只是传递了请求中断的消息*

*通常，中断是实现取消的最合理方式。*

{% highlight java linenos %}
public class PrimeProducer extends Thread {
    private final BlockingQueue<BigInteger> queue;
    PrimeProducer(BlockingQueue<BigInteger> queue){
        this.queue = queue;
    }
    public void run(){
        try{
            BigInteger p = BigInteger.ONE;
            while (!Thread.currentThread().isInterrupted()){
                System.out.println(p+"sleep 100 second.");
                Thread.sleep(100000);
                queue.put(p = p.nextProbablePrime());
            }
        }catch(InterruptedException consumed){
            System.out.println("out...");
            //允许线程退出
        }
    }
    public void cancel(){
        interrupt();
    }
    
    public static void main(String args[]) throws InterruptedException {
        BlockingQueue<BigInteger> queue = new LinkedBlockingQueue<BigInteger>();
        PrimeProducer p = new PrimeProducer(queue);
        p.start();
        Thread.sleep(1000);
        p.cancel();
    }
}
{% endhighlight %}

通过Future来取消任务

{% highlight java linenos %}
public static void timedRun(Runnable r, long timeout, TimeUnit unit) throws InterruptedException{
  Future<?> task = taskExec.submit(r);
  try{
    task.get(timeout, unit);
  }catch(TimeoutException e){
    //接下来任务将被取消
  }catch(ExecutionException e){
    //如果在任务中抛出了异常，那么重新抛出该异常
    throw launderThrowable(e.getCause());
  }finally{
    //如果任务已经结束，那么执行取消操作也不会带来任何影响
    task.cancel(true);//如果任务正则运行，那么将被中断
  }
}
{% endhighlight %}

`当Future.get抛出InterruptedException或TimeoutException时，如果你知道不再需要结果，那么就可以调用Future.cancel来取消任务。`

**“毒丸”对象**：指在一个放在队列上的对象，其含义是“当得到这个对象时，立即停止”。