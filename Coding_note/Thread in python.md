**Thread:** sequence of instructions in program that can be executed independently of the remaining process

Threads of a process can share the memory of global variables, if the variable's value is changed in one thread, it is applicable for all threads

2 types of threads:
- Deamon thread: daemon thread will shut down immediately when the program exits. 
- Non-deamon thread: If a program is running Thread that are not daemons, then the program will wait for those threads to complete before it terminates.

## Tutorial
- To start a separate thread, create a `Thread` instance and then tell it to `.start()`
- To tell one thread to wait for another thread to finish, you call `.join()`
- Working with multi threads:
```python 
import logging
import threading
import time

def thread_function(name):
    logging.info("Thread %s: starting", name)
    time.sleep(2)
    logging.info("Thread %s: finishing", name)

if __name__ == "__main__":
    format = "%(asctime)s: %(message)s"
    logging.basicConfig(format=format, level=logging.INFO,
                        datefmt="%H:%M:%S")

    threads = list()
    for index in range(3):
        logging.info("Main    : create and start thread %d.", index)
        x = threading.Thread(target=thread_function, args=(index,))
        threads.append(x)
        x.start()

    for index, thread in enumerate(threads):
        logging.info("Main    : before joining thread %d.", index)
        thread.join()
        logging.info("Main    : thread %d done", index)
```

- **ThreadPoolExecutor:**
```python
if __name__ == "__main__":
    format = "%(asctime)s: %(message)s"
    logging.basicConfig(format=format, level=logging.INFO,
                        datefmt="%H:%M:%S")

    with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
        executor.map(thread_function, range(3))
```

