# Starve-free-Readers-Writers-Problem
## Description
This repository contains the pseudo code to solve the classical Readers-Writers Synchronisation problem using semaphores where neither Readers nor Writers Suffer from Starvation

## Readers Writers Problem
The readers-writers problem is a classic synchronization problem in computer science that involves multiple threads (or processes) that compete for access to a shared resource, such as a file or database.

In this problem, there are two types of threads: readers and writers. Readers only need to read the shared resource, while writers need to modify it. The goal is to allow multiple readers to access the resource simultaneously while ensuring that only one writer can access it at a time to prevent race conditions.

## Common Solutions prone to starvation
Reader Priority: In this solution, readers are given priority over writers. If a reader is accessing the resource, then all subsequent readers can also access the resource, even if a writer is waiting to access it. This solution can lead to writer starvation if there are always more readers than writers, as writers may never get a chance to access the resource.

Writer Priority: In this solution, writers are given priority over readers. If a writer is waiting to access the resource, then all subsequent threads, including readers, must wait until the writer is done. This solution can lead to reader starvation if there are always more writers than readers, as readers may never get a chance to access the resource.

## Starvation free Solution

### Global Variables 
Following are the global variables and their initialization.
```C++
Semaphore* entry=1;     // allows a read of write porcess to start execution
Semaphore* exit=1;      // lock the access to shared variable reader_out
Semaphore* write_mutex=0;       // ensures that only one write is being executed
int reader_in =0;       // stores the total number of readers that have read the shared data
int reader_out=0;       //stores the total number of read operations that have completed execution
bool writer_available=0;        // flag variable that indicates if a writer is present in the waiting queue
```
### Readers Process

Following is the implementation of the readers process
```C++
Reader_process
{
    wait(entry);        //semaphore is used to ensure mutual exclusion on the shared variable reader_in 
    reader_in++;        // increase the number of read processes that will start to read
    signal(entry);      // allows multipple read operations to execute at once

    // Critical Section!!  Reading the data

    wait(exit);         // allow readers to exit sequentially and have mutual exclusion on the shared variable reader_out
    reader_out++;       // increment the number of readers that have finished reading data

    if(writer_available && reader_in = reader_out)
    {
        signal(write_mutex);  // the last reader must allow the next writer process to start writting
    }
    signal(exit);   //allow other readers to exit 
}
```
### Writers Process

Following is the implementation of the writers process
```C++
Writer_process
{
    wait(entry);    //the writer process acquires this semaphore to ensure that no 
                    //process whether read or write can start execution
    wait(exit);     // exit is acquired so that no read process can modify the value of reader_out used in the next line
    if(reader_in == reader_out)     // evaluates if all reader processes prior to the write process have exited their critical section
    {
        signal(exit);   //allows access to reader_out
    }
    else        
    {
        writer_available = true;    //writer waits until the last read operation has completed its execution
        signal(exit);               //allows access to reader_out for readers to exit
        wait(write_mutex);          // the last read instruction signals the writer process to start execution
        writer_available = false;   // flips that flag as it enters the critical section
    }

    // Critical Section!!  Writing the data

    signal(entry);  //releases the lock to allow other processes to enter next
}
```

## Explanation
This method allows any number of readers to enter the critical section simultaneously. Whenever a writer arrives, if there are still active readers 
then the writer sets the flag - writer_available to true and waits for the write-mutex semaphore. We have  managed to solve the problem of starvation due to the FIFO nature of the semaphore queues. 

If a writer arrives and there are still active readers then the writer simply waits until the write-mutex semaphore made available by the last terminating read operation.

Whenever a write process arrives, it acquires the semaphore- entry and holds it until it terminates, this ensures that no other write or read operations are overlapped with its execution. Now suppose more read or write operations arrive before our write operation had terminated. Then these operations will be blocked in the queue of the entry semaphore in the same order that they arrived in and so will be resumed using our FIFO policy. This ensures that no operation will be starved and synchronisation is maintained.








