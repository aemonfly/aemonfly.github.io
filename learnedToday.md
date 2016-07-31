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

