Design Document for Project 1: Threads
======================================

## Group Members

* May Cui <mayc@berkeley.edu>
* Kenneth Lautner III <klautner@berkeley.edu>
* Zeyana Musthafa <zeyana.a@berkeley.edu>
* Alan Song <alansong@berkeley.edu>

Replace this text with your design document.

## Part 1: Efficient Alarm Clock
#### Data Structures and Functions
Within timer.c:
``` C
/* Create a List of threads that will be sleeping as a variable in the file 
Note: Need to used list_insert_ordered to be able to have everything 
put in the correct order globally */
Static struct list sleeping_threads;

/* Use a lock for the sleeping threads so that there are no data races */ 
Static struct lock sleeping_threads_lock;
```
Within thread.c:
``` C 
/* For edge case you need to edit the thread struct */ 
Struct thread {
	Int64_t wakeup_time;
}

/* This means for the initialization of a thread needs to initialize wakeup_time. */ 
Static void init_thread(struct thread *t, const char *name, int priority) {
	...
	t->wakeup_time = 0;
}
```
#### Algorithms
The timer is supposed to have a thread sleep for a set amount of time. To deal with this without using excessive resources, you can  calculate when the thread will need to be awake (you are given the amount of ticks and the code given to us gets the starting time) and then acquire the `sleeping_threads_lock`.  After that we insert the thread into the `sleeping_threads` list using `list_insert_ordered()` so that the sleeping threads wake up in the correct order.  Furthermore we call `thread_block()` so that the thread doesn’t take up resource while in the `sleeping_threads` list.

Another thing to consider because of the lock is the `timer_interrupt()` function.  Within it we now need to check if the `sleeping_threads_lock` is held.  If it is not we take the threads from the `sleeping_threads` list that need to be woken up and wake them.

#### Synchronization
`sleeping_threads_lock` is used to prevent weird behavior where an interrupt occurs when `sleeping_threads` is being updated. This can lead to undefined behavior such as trying to pop items of the list that aren’t there. Timer interrupt never gets the lock (as it doesn’t need it) but checks if it is available.

`thread_block()` puts the threads that are put into the sleeping_threads list to sleep which means that they cannot be exited or deallocated until after `thread_unblock()` is called.  This is why it is safe to add it to the `sleeping_threads` list.

There is a possibility of there to be an interrupt before the thread is blocked. This means that there needs to be a sanity check with each thread to make sure this edge case is not occuring which is represented by `wakeup_time`.

#### Rationale
First and foremost, sorted linked lists are provided for us in the code. This greatly reduces the amount of coding we need to do. The time and space complexity for linked lists is pintos is `O(n)` to `O(1)` which is nice in terms of performance.The space complexity is just `O(n)`.



