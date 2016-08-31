# 2016-07-30
# 2016-07-31

## Synchronization Of Java 

Java **synchronized** has more meaning than exclusivly access, it also implies a way of communictaion
between threads. Saying that if one thread change the value enclosed by lock or synchronized keywords
will make sure all the thread seen the change of one thread get the lock.



``` java 

public class App {
    public static int shared = 0;
    
    pubic static synchronized setValue(int newValue) {
        shared = newValue;
    }
    
    //Thread1
       new Thread(new Runnable() {
           @Override
           public void run() {
               setValue(33);
           }
       }).start(); 

    //Thread2
       new Thread(new Runnable() {
           @Override
           public void run() {
               setValue(66);
           }
       }).start(); 
}


```

There two things happened when thread1 set the shared value as 33:

  * thread1 change the shared value as 33 internally in an atom operation.
  * thread1 use some way to notify the thread2 the new value of variable shared is 33. And thread2 is aware of the value change made by thread1

  * the jsr memory barriar make sure the notification by wrting the value back into main memory and invalidate the cache of other threads.
  
There has another java keywords  `volative` which is used to make sure the notification of value change and lack of the exclusively access
the field.

The jvm makes sure the write operation is atomic, beside the the long and double. And there is no worry about the dirty write of 
field of java, but probably has the dirty read because of the writing may not visible by other threads. it depends on many factor:
time, thread, lock and so on. 

And the volatile key words can make sure once the value of field is updated it can be seen by other thread.

So 

* jvm makes sure write operation atomic (beside long and double)
* volatile enhanced it by making the modification visible to related parties immediately.
* sychronized keywords or facility make the both.


## Thread failure ##

* if two thread can't access some fields exclusively, it will result in getting you in trouble, we call it unsafe failure.
* if two thread synchronized, but result in some dead lock or the thread can't advance, we call it thread liveness failure since none of them go forward.
* one thread can modify a object in a short time and only sync the shared object reference. After that point, we don't change the object anymore. the object is effectively immutable and one thread can publish the shared object to other threads safely, we call it safe publish.

There has many ways to do that: e.g 

* static field of class
* volatile field
* final filed 
* concurrent collection.



## CopyOnWriteList ##

Two members of CopyOnWrite(abbr as COW) containers: 

* CopyOnWriteArrayList
* CopyOnWriteSet

Of course we can easily writing out the COWMap.

### COW Principals

  * separate read from write. (simplified version of DB read/write split)

Threre is no lock when reading and need lock when writing which means it quit suitable for more read less writ sceniaros.

  * eventually consistent. Modification can't instantly reflected by other threads.

When one thread do the updating, it will copy the array entirely and made the change on the new array. And change the reference to new array when modification completed.

  * you can consider it when you need white/black list(serching keywords filtering).
  
  * Problems: it will almost double/trible the memory or even more.

### COW Implement ###

```java

    public COWContainer<T> {
        private ArrayList<T> list = new ArrayList<~>();
        public COWContainer() {
        
        }
        
        public T get () {
            return list.get();
        } 
        
        public boolean add (T element) {
            ReentryLock lock = this.lock;
            
            try {
                lock.lock();
                Object[] elements = getArray();
                
                Object[] newElements = Arrays.copyOf(elements, len + 1);
                
                newElements[len] = element;
                
                setArry(newElements);
                
                return true;
            } finally {
                lock.unlock();
            }
        }
    }
```
  
### Reference ###

[coolshell CopyOnWriteArrayList](http://coolshell.cn/articles/11175.html)

[CSDN CopyOnWritePrincipals](http://blog.csdn.net/zhangxs_3/article/details/8494675)

## Java Serialize interface 

## Misc ##

# 2016-08-01 #

## JMeter ##

JMeter is an opensource and powerfull tool as 

* load generator (simulate multiple user by mutliple thread or process)
* user run engine (simulate user behavior according to the script)
* Resource generator (generate the load test data when press test)
* reportor (create the report for user)

JMeters has the following core conceptions:

### TestPlan ###

TestPlan is the root of the jmeter test, it represents a test. You can define
Common user variables there and reference it from the subElement of the testPlan.

### Sampler ###

Sampler is an abstraction of request, its the core of the jmeter. A plenty of samplers inside the jmeter:

* HttpRequest
* HttpRestRequest rest sampler
* Ftp
* JDBC 
* Beanshell sampler

Beanshell sampler is most important one for very flexible usage because you can 
call do program there. its syntax is very similar to java except that omitting the variable type.

### Controller ###

Controller AKA logic controller, it will controll the flow of the test case. 
it simulate programmaing language unit:

* switch
* if
* loop (foreach, while)
* transaction (some kinds of synchronization). for example: all the request wait on somewhere and access the server simultaniously. it will act as atom, like the transaction in the database.

* misc (Once only(for login), Random, Random order, Runtime controller (??))


### Processor (pre/post)###

Before you make an request, you may want to modify some of the url parameters and want to extract the response field from the response body. e.g: get context form the http json response body. 

* BeansShell preProcessor/postProcessor (As a way of programming, you can do what ever)
* JSON Path Extractor (get json body or field)
* Result Action Handler. (do something according to the http status code)

### Assert ###

* when get response make sure the required condition is statified, just like unit test assert.
* 

### Timer ###

* Synchronizing time??

### Listener ###

It is widely used as result collector, for example: display the test result, draw all the performance charts.

### Variable And Function ###

Jmeter has a lot of buildin functions, you can reference it as ${__function} in the configuration.

### LifeCycle And Scope ###

the scope as its parent-child relation ship.

# 2016-08-02 #

## Queue ##

revisit the java queue, java provide two kinds of queue:

* Blocking queue
* Non-Blocking queue

### Blocking queue method vector:


|         | Throw Exception   | Return Boolean    | Maybe Blocked   | Block Wait Time   |
| ------- |:-----------------:|:-----------------:|:---------------:| -----------------:|
| Enqueue | add(e) :boolean   | offer(e) :boolean | put(e) : void   | offer(e,timeout)  |
| Dequeue | remove(e):boolean | poll(): E         | take(): E       | poll(e,timeout)   |
| Lookup  | elment() : E      | peek(): E         | N/A             | N/A               |


### Notes:

lookup methods (element and peek) just retrieves but not remove it from queue.


### Queue constrains:

* You can't add more element when queue is full. (You can sepcify the capacity of queue when creating it)
* You can't delete element when queue is empty.

So different method response differently when constrains are broken.

* Throw Exception (add, remove)
* Return boolean value to indicate success or not.
* Blocking current thread until it success.(constrains are met)

Blocking queue provides more flexibility, you can set timeout for operation(add or remove), which is frequently used
when programming.

*** Whatever adding method, it will throw nullPointerException if the parameter e is null.***

Impl Considerations:

* add() => offer() and throw illegalStateException if offer return false.
* remove() => pool() and throw NoSuchElementExceptions if poll return false.
* put/take use reentryLock synchronize the access and make the current thread wait on corresponding conditions.


### BlockingQueue category:

* ArrayBlockingQueue
* LinkedBlockingQueue
* PriorityBlockingQueue
* SynchronousQueue
* ScheduledThreadPoolExecutor DelayedWorkQueue 
* BlockingDeque and TransferQueue (interface)
* LinkedBlockingDeque impl BlockingDeque
* LinkedTransferQueue impl TransferQueue 

LinkedBlocking use two reentrantLock, putLock and takeLock to increase the concurrent threads over the queue.


### Non-Blocking queue:

* concurrentLinkedQueue
* concurrentLinkedDeque


### Other Concurrent Data Structure:

* ConcurrentMap & ConcurrentNavigableMap (interface)
* ConcurrentHashMap & ConcurrentSkipListMap
* ConcurrentSkipListSet


### concurrentLinkedQueue lock-free, wait-free analysis.

Basic Idea.

The lock will involve thread switch which is very heavy operation. the lock make sure two things: 

* Access the shared data structure exclusively.(implied that atom change and order)
* The modification is visible to other thread.

for lock-free, we can use ** volatile ** to assure the visiblity.

for atom change, we can use ** atomic variable ** to assure the field change's atom.

But for the order, we need to handle it carefully. for e.g:
when insert a node into the link, we firstly insert the new node into the link end and then make the tail pointing the new node.

Without lock, many threads can do any step of the above two steps, we need a way to make sure the thread cooperate with each other well. 

Say, two thread try to insert new nodes into link at the same time. Both of them will insert the new code of themselves to the end of the link, obviously the last modification will win, but it is not what want. We need way to make sure threads aware of other thread's existence, and work together with them.

The easyiest way maybe detect the change of current data structure, say if tail's next has been change by others, if the tail is the real tail. As for work together, for link, we can help advance the tail to the real tail of current link.

Code Fragment as belowing:

```
Item newElement = e;

Node<Item> t = tail;

//we can't use tail.next() here
//since tail get change right after we assign the tail to t even though it is
//very low probably.

Node<Item> s = t.next();

//update tail's next

    if( t == tail) {
        if (s == null) {
            if(t.casNext(s, n)) {
                casTail(t,n);
                return true;
            }
        } else {
        // help avance the tail 
            casTail(t, s)
        }

    }

```

More notes about casTail and casNext, it use the hardware primitives compareAndSet to make sure the change operation's atom.


### Atomic Variable ###

the java util package has some atomic variables: such as

* AtomicReferenceFiledUpdater
* AtomicLong
* AtomicIntegrate


### Resource Reference ###

[java thread safe queue summary](http://hellosure.iteye.com/blog/1126541)

[java wait notify misc](http://blog.csdn.net/tayanxunhua/article/details/20998809)

[java memory model](http://lujinxiong.blog.51cto.com/9367514/1786558)

[java non-block algrithom introduction](http://www.ibm.com/developerworks/cn/java/j-jtp04186/)

        
# 2016-08-03 #
# 2016-08-04 #


### SynchronousQueue ###

synchronousQueue is special queue, its size is zero. You can't peek its element, it behavior as following:

* isEmpty => always true
* size() => 0
* peek() => null
* element() => throw NoSuchElementException() 

it is used to sync two threads exchange data. When one thread try to put, if no thread are waiting for consuming,
the put thread will be blocked until some one to take the data away.

The producer and consumer wait for each other, and exchange data and then say goodbye to each other. its behavior 
really like cyclicBarrier, the thread wait for each other. But synchronousQueue do more, exchange data.

### SynchronousQueue Impl ###

The easiest way is use reentrylock or synchronized keywords.

The second way is use semaphor. Use three semaphors: send(1), recv(0), sync(0).

When putting item:


```
    send.acquire();
    saveItem(element);
    //notify consumer
    recv.release();
    sync.acquire();
```

when taking item:


```
    recv.acquire();
    Element e = getItem();
    sync.release();
    send.release();
```

the JDK1.5 use the AbstractQueueSynchroinzer to implement the synchronousQueue, And we will 
talk the AbstractQueueSynchronizer later.

the JDK1.6 use lock-free algrithm, which better performance over jdk1.5.

### TransferQueue LinkedTransferQueue ###

The JDK1.7 adds new queue named transferQueue, according to the Dog Lea's speaking:

Its an blockingQueue, concurrentLinkedQueue, synchronousQueue. It supports the following:

* Act as BlockingQueue, you can use the blockingQueue interface method.
* Act as SynchronousQueue. it add new method transfer, tryTransfer, tryTransfer(timeout).it has the 
semantics of add the element to the queue and block the thread until the elment being consumed by some
other thread. 

* the linkedTransferQueue has more efficient implementation. 
* it has two aided methods, hasWaitingConsumer()  & getWaitingCounsumerCount()


### SynchronousQueue Usage ###

Inside the jdk, the Executors.newCachedThreadThreadPool() uses it. it create new thread when
new task arriving and reuse the existing free thread if possilbe. the thread will be killed if free for 60 seconds.


### Resource Reference ###

[Java Synchronous Queue](http://ifeve.com/java-synchronousqueue/)

[Java7 Transfer Queue](http://ifeve.com/java-transfer-queue/)
