# (Notes from Modern C++ (cpp) Concurrency - Mike Shah - https://www.youtube.com/playlist?list=PLvv0ScY6vfd_ocTP2ZLicgqKnvq50OCXM)

**Parallelism: Everything happens at once, instantaneously, completely separate.**

**Concurrency: Multiple things can happen at once, sometimes tasks have to wait on shared resources.**

Create multiple threads (on the same core) and use std::mutex to prevent data race.

For instance;
std::mutex can be manually locked and unlocked 

	std::mutex lock;  //creation of a mutex
	
	void shared_value() {
		lock.lock();
		some code…
		lock.unlock(); 
	} 

or we can use lock_guard at the beginning of the function:

	std::lock_guard<std::mutex> lockGuard(lock);

or use atomic variables to make accessing shared data(primitive types) safe:

	std::atomic<int> name; 

Use condition_variable to prevent threads from waiting to access data and wasting CPU cycles. We need a boolean, condition_variable and lock to implement this:
		
	std::mutex lock;  
	std::condition_variable c_variable;
	bool notified = false;
			
	Reporter thread {
				std::unique_lock<std::mutex> uniqueLock(lock);
				if(!notified) 
				c_variable.wait(lock);
				do something..
	}	
	
	Worker thread {
				std::unique_lock<std::mutex> uniqueLock(lock);	
	do something…
				notified = true; 
				c_variable.notify_one();
	}	

std::async: Launch a thread that will not be blocked.
std::future: It is used to access the results that async operations produce.

Basic syntax;

	std::future<int> asyncFunction = std::async(std::launch::async, &func, Args);
			some code….
			Int result = asyncFunction.get(); // We wait until the function is complete.
		
For example;

	#include <iostream>
	#include <future>
	#include <chrono>
	
	bool task1 (int x) {
	    for(int i = 0; i < x; i++) {
	        	std::cout << "loading\n";
	       	std::this_thread::sleep_for(std::chrono::milliseconds(x));
	    }
	    std::cout << "Loading completed\n";
	    return true;
	}

	int main() {
	
	std::chrono::time_point start = std::chrono::high_resolution_clock::now();
	std::future<bool> control = std::async(std::launch::async, &task1, 20);
	std::future_status status;
	
	while(true) {
		 std::cout << "Main is running\n";
			 status = control.wait_for(std::chrono::milliseconds(1));
		 if(status == std::future_status::ready){
					std::cout << "Program is done\n";
					break;
			}
	}
	std::chrono::duration<double> eTime = std::chrono::high_resolution_clock::now() - start;
	std::cout << "Elapsed time: " << eTime << "\n";
	return 0;
	}

https://en.cppreference.com/w/cpp/thread
