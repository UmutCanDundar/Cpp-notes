
# WHAT IS THREAD?

Threads are parts of the process. We can create thread objects by using std::thread and pass callable objects(function pointers, function objects, lambdas, member functions) into them. 

# WHAT ARE SINGLE-THREADED AND MULTI-THREADED PROGRAMMING?

If threads execute in order, it is called single-threaded programming, which means that one thread waits to finish another thread before it runs; in contrast, detached threads run independently from other threads, if we divide one program into two or more separate parts by using detached threads, it is called multi-threaded programming.

We use join() and detach() methods to determine which way a thread works. 

# WHAT ARE MUTEX AND DEADLOCK?

Mutex prevents separate threads which share the same data resources from accessing the data at the same time.

In concurrent computing, if each thread waits for another waiting process to proceed, it is called deadlock because the program cannot make any progress.
