# 2016-07-30

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
* if two thread synchronized, but result in some dead lock, we call it thread liveness failure since none of them go forward.
* one thread can publish object to other threads successfully , we call it safe publish.


## CopyOnWriteList ##



## Java Serialize interface 

## Misc ##

