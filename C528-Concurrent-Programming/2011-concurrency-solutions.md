### 1 

a.

Non-deterministic choice can be represented in FSP by the '|' symbol. This means either the action before the symbol could have occured or the action after.

This is important because in some systems it is impossible to predic the order of actions and we need to model both possibilities.

b.i.

```
TALK = (arrive -> front -> TALK
       |begin -> BEGUN),
BEGUN = (arrive -> back -> BEGUN).
```

b.ii.

```
NEVER = (arrive -> ERROR
	|on -> ERROR
	|time -> ERROR).
```

b.iii.

```
LCHAN = (in -> out -> LCHAN
	| disconnect -> STOP 
	| in -> LCHAN).
```

b.iv.

```
CUBE = CUBE[0],
CUBE[i:0..5] = (when i < 4 [i*i*i] -> CUBE[i+1]
			|when i ==4 [i*i*i] -> STOP).
```


c.i.

```
OVERFLOW = CAPACITY[0],
CAPACITY[i:0..3] = (drip -> CAPACITY[i+1]).
```

c.ii.

```
BIKE = (start->(stop->STOP | faster->slower->BIKE)).
```

c.iii.

```
BLOCK = STOP + {north}.
TRAIN = (north -> move -> TRAIN | south -> move -> TRAIN).

|| SYS = (BLOCK || TRAIN).
```

c.iv.

```
EXIT = (in->out -> EXIT).
|| OR = (a:EXIT || b:EXIT) /{in/{a.in, b.in}}.
```


### 2

a.i.

```
TEA_ROBOT = (sugar_please[i:1..3] -> SUGAR[i]),
SUGAR[i:1..3] = (when i > 1 sugar -> SUGAR[i-1]
		|when i ==1 sugar-> TEA_ROBOT).
```

a.ii.

```
property PROPER_TEA = (milk -> tea -> (drink -> PROPER_TEA | sugar -> drink -> PROPER_TEA)) + {slurp}.
```

b.i.

```
SAFE = (sip -> (tooHot -> SAFE | hot -> drink -> SAFE)).
```

b.ii.

```
property SAFE = (sip -> (tooHot -> SAFE | hot -> drink -> SAFE)).
```

The difference is that part ii. is defined as a property and therefore will specifically not allow any action to occur out of the order specified, and will label their terminal states as errors.

c.i.

notify() wakes up a single thread that is waiting in this object's waitset.

notifyAll() wakes up all threads that are waiting on this object's waitset.

c.ii.

Use notify() if all your waiting threads are interchangeable (the order they wake up doesn't matter), or if you only ever have one waiting thread. A common example is a thread pool used to execute jobs from a queue--when a job is added, one of threads is notified to wake up, execute the next job and go back to sleep.

Use notifyAll() for other cases where the waiting threads may have different purposes and should be able to run concurrently. An example is a maintenance operation on a shared resource, where multiple threads are waiting for the operation to complete before accessing the resource.

If you should use notifyAll() but end up using notify() instead, you may not wake up the thread that needs to be woken up and thus you would have the possibility of causing deadlock.

### 3

a. 

```
ALLOC = OPEN[0],
OPEN[i:0..MAX] = (when i > 0 get -> OPEN[i-1]
		|when i < MAX put -> OPEN[i+1]
		|close -> CLOSED[i]),
CLOSE[i:0..MAX] = (service -> OPEN[i],
		when i < MAX put -> CLOSE[i+1]).


||COMMONROOM = ({xavi,yael,zak}:MEMBER||wally:TECHNICIAN||saucer:ALLOC||cup:ALLOC).

```

c. 

```
public class Allocator{

	int capacity;
	int used = 0;
	
	public void build(int n){
		capacity = n;	
	}
	
	public synchronized void get()
		throws InterruptException{
		
		while(used>= capacity) wait();
		used++;
		notifyAll()
	}	
	
	public synchronized void put()
		throws InterruptException{
		used--;
		notifyAll()
	}
}


public class Member{
	
	public void get (Allocator a){
		a.get();
	}
	
	public void put (Allocator a){
		a.put();
	}
}


```

c.

```
xavi.cup.get
wally.cup.close
wally.saucer.close

// Xavi is waiting to get the saucer but can't because the saucer is closed. Wally is waiting for all the cups to be returned to service the machine but Xavi won't give up his cup without getting a saucer. A modern tragedy.
```


### 4 

a.

class MyThread extends Thread { … } Thread a = new MyThread();

Manages a single thread; threads can be created and deleted dynamically.

class MyRun implements Runnable { … } Thread b = new Thread(new MyRun());

Java does not allow multiple inheritance, so sometimes it’s better to implement interface Runnable instead.

b. 


```
class MyClass {
	private ResourceA a = new Resource();
	private ResourceB b = new Resource();
	
	private Object lockA = new Object();
	private Object lockB = new Object();
	
	public void updateA (int x){
		synchronized(lockA){
			a.update(x);
		}
	
	}
	
	public void updateB (int y){
		synchronized(lockB){
			b.update(y);
		}
	
	}
}

```

d.

“We discuss deadlock in more detail in the next chapter. However, in essence, it means that a system can make no further progress since there are no further actions it can take. The deadlock in the model can be seen in the demonstration version of the program by starting the consumer and letting it block, waiting to get a character from the empty buffer. When the producer is started, it cannot put a character into the buffer. Why? The reason is to do with the use of two levels of synchronization lock: the first gives mutually exclusive access to the buffer monitor and the second gives mutually exclusive access to the semaphores.”

“When the consumer calls get, it acquires the Buffer monitor lock and then acquires the monitor lock for the full semaphore by calling full.down() to check if there is something in the buffer. Since initially the buffer is empty, the call to full.down() blocks the consumer thread (using wait()) and releases the monitor lock for the full semaphore. However, it does not release the monitor lock for Buffer. Consequently, the producer cannot enter the monitor to put a character into the buffer and so no progress can be made by either producer or consumer – hence the deadlock. The situation described above is known as the nested monitor problem.”

Excerpt From: byJeff MageeandJeff Kramer. “Concurrency&#8212;State Models &amp; Java Programs, 2nd Edition.” iBooks. 


