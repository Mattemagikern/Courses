# EDAN26
Author: Måns Ansgariusson, C14

## Multicores 
**Definition of multicore** - a core is an independent processor. When a chip
has multiple cores it's called a multicore. 

**Definition of multiprocessors** - a parallel computer in which threads can
access a shared memory using normal load and store instructions. 

## Amdalh's law
Amdahl's law is a formula which gives the theoretical speedup in latency of
the execution of a task at fixed workload that can be expected of a system
whose resources are improved.

1/((1-p) + p/N)

## Cache-coherence
In a shared memory multiprocessor system with a separate cache memory for each
processor, it is possible to have many copies of shared data: one copy in the
main memory and one in the local cache of each processor that has requested the
data. When one of the copies of data is changed, the other copies must reflect
that change. Cache coherence is the discipline which ensures that the changes
in the values of shared operands(data) are propagated throughout the system in
a timely fashion.


**requirements**:

* **write propagation** - Changes to the data in any cache must be propagated
to other copies(of that cache line) in the peer caches.

* **transaction serialization** - Reads/Writes to a single memory location must
be seen by all processors in the same order.  


**Cold Miss or compulsory miss** - This is the first access to a block which
MUST result in a cache miss. To be EXTREAMLY clear: When we want to access a
memory block we try to find it in the cache, to no surprise (if we haven't
requested it before) we won't find it. Then the memory block is fetched from
the primary or secondary memory and placed in the cache. There of the name
compulsory miss, if it hasn't been in the cache memory before we need to fetch
it => cache miss. 


**Capacity miss** - When you iterate over a data structure larger than the
capacity of the cache the resulting misses except the first access of each line
which is a compulsory miss is capacity misses. 


**Conflict miss** - In the case of set associative or direct mapped caches,
conflict misses occur when several blocks are mapped to the same set or block
frame in the cache.


**coherence misses**: Cache misses that are results of shared memory structures
between processes are called cache coherent misses. There is two types of cache
coherent misses; True sharing miss and false sharing miss. These can ,perhaps
surprisingly, happen when the computer uses sequential consistency models. 


**True sharing misses** - True sharing cache misses occur whenever two
processors access the same data. True sharing requires the processors involved
to explicitly synchronize with each other to ensure program correctness. This
happens whenever a process changes a shared data structure and in doing so sends
an invalidation flag of the cache block to all the other processes, then when
an other process want to read the value stored in the cache block it needs to
fetch it from the memory to ensure the program correctness. 


**False sharing misses** - False sharing results when different processors use
different data that happen to be co-located on the same cache line. Even if a
processor re-uses a data item, the item may no longer be in the cache due to an
intervening access by another processor to another word in the same cache line.
This happens due to the processors inability to predict the future. Because it
doesn't know which data word in the cache block that has been invalidated it
needs to fetch the entire cache block from the memory, even though the data
word it was interested in hasn't been invalidated by another process. 

**Cache states**:
  * Shared: This cache and possibly others have a copy of this memory block.
  * Invalid: This cache's copy is no longer valid, a value in the cache block
    has been modified since the previous load instruction. 
  * Modified: This cache owns the cache block and has modified it, this
    notifies the cache to send the update to the main memory.
  * Exclusive : This cache owns the cache block and can modified it but hasn't. 

## POSIX - Threads in C 
POSIX,Portable Operating System Interface for Unix, is a family of standards
specified by IEEE to clarify and make uniform the application programming
interfaces (and ancillary issues, such as commandline shell utilities) provided
by Unix-y operating systems. When you write your programs to rely on POSIX
standards, you can be pretty sure to be able to port them easily among a large
family of Unix derivatives.


### Pthreads 
pthreads is a thread management interface in the POSIXs standard, it defines how
to create threads, the use of mutexes, conditions variables and
synchronization between threads. 

#### Undefined behavior
Undefined behavior is the result of executing computer code whose behavior is
not prescribed by the language specification to which the code adheres, for the
current state of the program (e.g. memory). This happens when the translator of
the source code makes certain assumptions, but these assumptions are not
satisfied during execution. An example of undefined behavior is:


```c
int main(){ 
    int array[10]; 
    for(int i = 0; i < 10; i++) 
      array[i]=i;

/* here well go outside the scope of our data resulting in undefined
behavior, becuase we won't know whats printed at the end of the array
(element 11)
*/
    for(int i = 0; i =< 10; i++)   
      printf("%d ", array[i]);

  return 0; 
}

``` 

#### Data Races 
A race condition occurs when two or more threads can access shared data and
they try to change it at the same time.  Because the thread scheduling
algorithm can swap between threads at any time, you don't know the order in
which the threads will attempt to access the shared data.  Therefore, the
result of the change in data is dependent on the thread scheduling algorithm,
i.e. both threads are "racing" to access/change the data. 

      
A memory rule to avoid data races is: "Any time two threads operate on a
shared variable concurrently, and one of those operations performs a write,
both threads must use atomic operations."

#### Mutual exclusion && Mutex && condition variable 
Mutual exclusion is a property of concurrency control, which is instituted for
the purpose of preventing race conditions; it is the requirement that one
thread of execution never enter its *critical section* at the same time that
another concurrent thread of execution enters its own critical section. To
solve this the POSIXs standard provide **Mutex**. 


**critical section** - concurrent accesses to shared resources can lead to
unexpected or erroneous behavior, so parts of the program where the shared
resource is accessed are protected. This protected section is the critical
section or critical region. It cannot be executed by more than one process.
Typically, the critical section accesses a shared resource, such as a data
structure, a peripheral device, or a network connection, that would not operate
correctly in the context of multiple concurrent accesses. 


**Mutex** - is to be viewed as a lock for data structures or critical sections.
When the mutex is active it prevents multiple threads to access a critical
section which guarantees the integrity of the shared resource(s). Mutex is a lock
with a sleep queue(opposite to a spin lock).
Mutex is predefined in the C language in the ``<threads.h>`` (c11) or ``<pthreads.h>``
(C99). Mutex is a great way to illustrate how release consistency works, by
``pthreads_mutex_lock(&lock);`` we acquires the lock and by
``pthreads_mutex_unlock(&lock);`` we release the lock, guaranteeing the
correctness of the code inside the mutex block. 


**Condition variable** - 	While a mutex lets threads synchronize by controlling
their access to data, a condition variable lets threads synchronize on the
value of data. Cooperating threads wait until data reaches a particular state
or until a certain event occurs. Condition variables provide a kind of
notification system among threads. As mentioned earlier, if Pthreads didn't
offer condition variables, but only provided mutexes, threads would need to
poll the variable to determine when it reached a certain state.

 
**Intercepted wakeups** -  a thread which owned the mutex and signals the
condition variable can either unlock the mutex first or call signal first.  If
the mutex was unlocked first (which can reduce the number of context switches)
a third thread can take the lock before the signalling occurs and this is
called an intercepted wakeup.  Since the third thread can change the state the
first thread may need to wait again. Hence a loop is needed.  


**loose predicates** - a predicate which indicates the woke up thread might
have something to do and if it turns out it hasn't anything to do it should
continue waiting until woken again.


**spurious wakeups** - Even after a condition variable appears to have been
signaled from a waiting thread's point of view, the condition that was awaited
may still be false. One of the reasons for this is a spurious wakeup; that is,
a thread might be awoken from its waiting state even though no thread signaled
the condition variable. For correctness it is necessary, then, to verify that
the condition is indeed true after the thread has finished waiting. Because
spurious wakeup can happen repeatedly, this is achieved by waiting inside a
loop that terminates when the condition is true, for example: 


```c 
  /* In any waiting thread: */ 
  while(!buf->full) 
    wait(&buf->cond, &buf->lock);

  /* In any other thread: */ 
  if(buf->n >= buf->size){ 
    buf->full = 1;
    signal(&buf->cond); 
    } 
``` 

##### Memory fences
A memory fence is a mutex inhibiting the compiler to perform optimizations
before and after certain points in the code.


##### Busy-wait (Spinlock) 
Busy-wait or spinning is a technique in which a process repeatedly checks to see
if a condition is true, such if a lock is available.  Spinning can be a valid
strategy in certain circumstances, most notably in the implementation of
spinlocks within operating systems designed to run on SMP systems. In general,
however, spinning is considered an anti-pattern and should be avoided, as
processor time that could be used to execute a different task is instead wasted
on useless activity.


**Futex** - ``<fast mutex>``, an implementation to use when working with quick
operations.works by using the spinning principle.

#### Deadlocks 
Deadlock is a state in which each member of a group is waiting for some other
member to take action, such as sending a message or more commonly releasing a
mutex.  For a deadlock to arise the computer system needs to fulfill the
following conditions: 

* **Hold and wait** - a process is currently holding at least one resource and
  requesting additional resources which are being held by other processes.

* **No preemption** -  a resource can be released only voluntarily by the
  process holding it.

* **Mutual exclusion** - The resources involved must be unsharable; otherwise,
  the processes would not be prevented from using the resource when necessary.
  Only one process can use the resource at any given instant of time.

* **circular wait** - each process must be waiting for a resource which is
  being held by another process, which in turn is waiting for the first process
  to release the resource.  
  
## Memory Consistency models 
  
### Atomic 
An operation acting on shared memory is atomic if it completes in a single
step relative to other threads. This means that a load and write operation to
a memory cell is done without interruption from another thread.

**How do we detect a write completion?**: Because most computers have several
cache memories it is unclear when it is safe to read the new value, because the
caches can have an outdated version of the value stored in them. This problem
is solved by the memory sends a invalidation flag to the cache memories and
when they have acknowledged the invalidation of the memory the memory sends a
acknowledge to the original process to say that the write was successful and
the other processes are clear to proceed.

**Write access penalty** - This penalty is due to the fact that the processor
cannot perform another memory access before the previous has completed. Using
relaxed memory models you can lessen the perceived latency of the cache-miss by
processing other read/writes while the cache is updating. 

### Sequential consistency 
In systems such as these, the execution is deemed correct if the result is the
same as one in which the statements were executed in program order. A processor
satisfying this requirement is said to be sequential.  **requirements**:
* Writing atomically, all writes to the system is visibly to every other
  process before another fetch instruction aimed at the shared memory is
  started by a process.

* Sequential program order, The operations of all the processors had been
  executed in some sequential order.

The main disadvantages of Sequential consistency is the write access penalty
due to the processor cant handle an other memory access until the previous has
been completed and limiting of the compiler and hardware optimization. 

#### Dekker's algorithm
Dekker's algorithm is the first known correct solution to the mutual exclusion
problem in concurrent programming. It allows two threads to share a single-use
resource without conflict, using only shared memory for communication.

If two processes attempt to enter a critical section at the same time, the
algorithm will allow only one process in, based on whose turn it is. If one
process is already in the critical section, the other process will busy wait
for the first process to exit. This is done by the use of two flags,
wants\_to\_enter[0] and wants\_to\_enter[1], which indicate an intention to
enter the critical section on the part of processes 0 and 1, respectively,
and a variable turn that indicates who has priority between the two
processes.
Mutual exclusion will still be guaranteed as neither process can become
critical before setting their flag.

**Dekker's algorithm only works with Sequential consistency**. 
#### increment
An example of post increment:
```c
x ++; 
```
An example of pre increment: 
```c
++x; 
```
Both of the increments have sequential consistency because of a compiler
implementation. However they differ in when the increment is taking place. As
it is written now both will result in the same behaviour but if they were to
be applied in a function call the pre increment would be conducted before
entering the function and the post increment would be applied just as the
function hits its return statement. 

If it were to change the code to: 
```c
x = x + 1; 
```
Then we would untangle the load and store variable which would in turn allow
for better compiler optimization regarding load and store. 

### Transactional memory model.
Transactional memory attempts to simplify concurrent programming by allowing a
group of load and store instructions to execute atomically. Transactional
memory systems provide high level abstraction as an alternative to low level
thread synchronization. This abstraction allows for coordination between
concurrent reads and writes of shared data in parallel systems.

Transactional memory works using several instructions: 
* Read-Set: The set of data items (memory locations) that are read by a
  transaction.

* Write-Set: The set of data items (memory locations) that are written by a
  transaction.

* Commit: When a transaction successfully completes, we say that the
  transaction commits. When the transaction commits, all new values for the
  data items in the transaction’s write-set are made visible to the rest of
  the system.

* Abort: Abort means that the transaction fails, usually as a result of a
  conflict. When a transaction aborts it must restore its initial state, i.e.,
  reset all data items in the transaction’s write-set to the value they had
  when the transaction began.

* Conflict: Two concurrent transactions is said to conflict if one
  transaction’s write-set overlaps with the other transaction’s read or
  write-set. In case of a conflict, one of the transactions needs to abort.

A transaction generally has **three phases**:
1. Begin the transaction. A snap-shot of the execution state is taken, which
   will be needed if the transaction is aborted.

2. Execution of one or several operations/actions/tasks (the terminology
   varies). The effects of these operations are not visible outside the
   transaction during the execution of the transaction.

3. Commit or abort the transaction. In case of a commit, the result of the
   transaction and its associated operations are made visible the
   rest of the system. In case of an abort, the execution rollback's to
   the start of the transaction.

In transactional memory systems, a transaction is a finite code sequence that
satisfies the atomicity and isolation properties:

* Atomicity - Either the whole transaction is successful (committed) or none of
  it is done (aborted).

* Isolation - Individual memory updates within an ongoing transaction is not
  visible outside the transaction. When the transaction commits, all memory
  updates are made visible to the rest of the system.

### Relaxed memory model 
To increase speed and decrease idle and waiting time, modern high speed
processors often execute operations in a different order than which is
specified by the program. This reduces write penalties A LOT or removes them
entirely. Which allows for greater speedups. The sequential consistency model
doesn't allow reordering of data instructions but memory consistency models like
weak ordering and release consistency does not have the same restraint, making
it possible for greater speedups and better use of parallelism. 

In Java, C, C++ we use relaxed memory models when executing.

#### Weak ordering 
Weak ordering is the first Relaxed memory models. Weak ordering is a memory
consistency model that introduces a sync instruction. The sync instruction
guarantees that all instructions above has finished. Weak ordering assumes
shared data is modified in critical sections IE sections surrounded by sync
instructions. The compiler can change the order of operations inside a sync
block and around the sync block but not take operations outside the sync and
bring it in. 

To clarify: The sync instructions guarantees that everything before
that instruction is finished, every write to the memory has received an ack from
the memory saying the update has been made. This memory model works the same as
it would in C,C++ or Java. The critical sections needs to be surrounded by
mutex. The sync instructions are there to grantee the programs integrity and
to make it possible to gain speedups by changing the way the CPU write and read
from the memory.

In a  multiprocessor system, storage accesses are weakly ordered if:

1. Accesses to global synchronizing variables are strongly ordered.
2. No access to a synchronizing variable is issued by a processor before all
   previous global data accesses have been globally performed.
3. No access to global data is issued by a processor before a previous access
   to a synchronizing variable has been globally performed.

#### Release consistency 
Release consistency is an extension of the weak ordering model, it allows for
acquiring and releasing locks. When a lock is acquired a process "locks" all
read/write operations from different processes, in or about to enter the
critical section, until the thread releases the lock.  

Several processes can be on the acquire state and wait for the lock, there is no
way telling which process will get the lock. Due to it is up to the computers
scheduling of the processes. 

There can be several locks but the locks can't be used for the same
data structure without the possibility of a data-race. 

Writes before a lock can be processed while the process is in the
acquire-release block but the process isn't going to release the lock until
all writes has been given an ack from the memory.

The release consistency model is a very aggressive memory model and lets the
compiler to change the order of:

* Read and writes (data and data).
* Data and acquire
* Release and data
* Release and acquire (Highly unique situation when this option is applied but
  it is possible)

#### Sequence points
A sequence point is a point in time at which the dust has settled and all side
effects which have been seen so far are guaranteed to be complete. When you
reach a Sequence point it is guaranteed that the effect and side effects has
been completed before continuing past the Sequence point.

**An example of effect and side effect:**</br>
The statement: ``f(i++)`` has both an effect as in the value of i being copied
to the function f input parameter and the side effect that the value of i
is incremented. There is an important part to notice: The input parameter to a
is the value of i before being incremented. Given that i = 0 before the
statement ``f(i++)``, when the program enters the function f, **f's input
parameter will be 0** and the ``variable i`` will be 1. 

In C and C++ sequence points occurs in the following places:

1. Between evaluation and of the left and right operands of the &&, ||, and
   comma operators. Ex: ``*p++ != 0 && *q++ != 0`` , all side effects of the
   sub-expression  ``*p++ != 0`` are completed before any attempt to access q.

2. Between the evaluation of the first operand of the ternary ?: operator. This
   could lead to logical bugs, An example: ``b = (*p++) ? (*p++): 0``, A very
   stupid way of coding but bare with me, this expression would first increase
   the value stored at p before entering the second ``(*p++)``. 

3. At the end of a full expression ``;``, return statements and expressions
   inside the logical part of a if, for, while, swich and do-while. Ex:
   ``if(logical part)``, ``for(logical parts)`` and so on. 

4. Before entering functions. Ex. ``f(i++)``. You might think that the input
   parameter were to be executed before entering the function but you are
   incorrect! It will send a copy of i to the function f and then performed
   the operations and after it is done it will increase the value of i of one.
   This is a especially hard statement.  

A great example of ambiguous results due to the sequence points above is:


```c
void f(int a){
    ....
}

void g(int a){
    ....
}

int main(){

  int i = 1;
  i = f(i++) + g(i++); // Note: this is an ambiguous expression.
  printf("%d \n", i);

  return 0;
}
```
**So why is the statement ambiguous?**
It is ambiguous because it is not specified in the C standard how the
expression is going to be modified. Since we don't have a sequence point we
can't be sure which statement has been performed first. A compiler may
evaluate the g(i++) function before the f(i++) function.

**A great remark to make**: The if you were to implement the function above
and compile it in GCC you would not receive any errors. However compiled in
Clang you would receive a warning regarding the ambiguous statement, as can be
seen below. 
```
warning: multiple unsequenced modifications to 'i' [-Wunsequenced]
```
### Direct Memory Access (DMA) 
**DMA** is a feature of computer systems that allows certain hardware
subsystems to access main system memory (Random-access memory), independent of
the central processing unit (CPU). Without DMA, when the CPU is using
programmed input/output, it is typically fully occupied for the entire duration
of the read or write operation, and is thus unavailable to perform other work.
With DMA, the CPU first initiates the transfer, then it does other operations
while the transfer is in progress, and it finally receives an interrupt from
the DMA controller when the operation is done. This feature is useful at any
time that the CPU cannot keep up with the rate of data transfer, or when the
CPU needs to perform useful work while waiting for a relatively slow I/O data
transfer.

DMA requests are tagged with an integer in the range 0->31 and the SPU can wait
for all pending requests with a certain tag value. This way, by using different
tags for different loop iterations (for example) the programmer can achieve
concurrency within an SPU since both computation and data transfers are
happening concurrently.  

### Cell processor
The Cell processor is a powerPc processor developed by Sony, Toshiba and IBM as
a possible replacement of todays x86-architecture. The Cell processor is today
implemented in the sony playstation 3 as main processor. The Cell processor is
a 8-core processor capable at reaching 10x the speed of conventional
processors. 
Consists of one multithreded powerC: processor, eight simd vector spu,one
programmable memory interface controller, two input/output interfaces.  If
programmed cleverly the cpus can achieve very high performance. The PlayStation
3 is an example of a computer using the Cell processor. 


#### SPU (Synergistic Processor Unit) 
The SPU are optimized for SIMD vector processing and concurrent data transfers.
The SPU have no caches and both data and instructions must be explicitly
transferred from system memory by software using DMA.
The SPU is a processing unit besides the CPU that works on specific.(?)  

#### SIMD (Single instruction, multiple data) 
SIMD is a hardware optimization or parallel computing for microinstructions
mainly done by the ALU and the FPU. The short version is that the CPU can
simultaneously calculate arithmetic operations thanks to several ALUs and FPUs
if the operands is a so called SIMD vector. 

The long version is that the CPU is presented with two vectors whith
corresponding values to perform an operation on. 

Ex:

```c
SIMD1 = [1,3,4,7,9]
SIMD2 = [1,2,3,4,5]
=>
[2,5,7,11,14]
```

All these additions are performed simultaneously by the CPUs ALUs (if the CPU
has enough ALUs). By exploiting this the CPU uses 1(!!!!!) clock cycle to
perform 5 additions. That is truly amazing!

## Tools 
### Valgrind 
Is a generic framework for creating dynamic analysis tools such as checkers and
profilers. Memcheck is the default tool for valgrind. The problems Memcheck can
detect and warn about include the following:
  * Use of uninitialized memory
  * Reading/writing memory after it has been free'd
  * Reading/writing off the end of malloc'd blocks
  * Memory leaks The price of this is lost performance. Programs running under
    Memcheck usually run from twenty to thirty times slower than running
    outside Valgrind and use more memory.

#### Helgrind
Helgrind is a Valgrind tool for detecting synchronisation errors in C, C++ and
Fortran programs that use the POSIX pthreads threading primitives.


The main abstractions in POSIX pthreads are: a set of threads sharing a common
address space, thread creation, thread joining, thread exit, mutexes (locks),
condition variables (inter-thread event notifications), reader-writer locks,
spinlocks, semaphores and barriers.


Helgrind can detect three classes of errors, which are discussed in detail in
the next three sections:


* Misuses of the POSIX pthreads API.

* Potential deadlocks arising from lock ordering problems.

* Data race(s) - accessing memory without adequate locking or synchronisation.


Problems like these often result in unreproducible, timing-dependent crashes,
deadlocks and other misbehaviour, and can be difficult to find by other means.

Helgrind is aware of all the pthread abstractions and tracks their effects as
accurately as it can. On x86 and amd64 platforms, it understands and partially
handles implicit locking arising from the use of the LOCK instruction prefix.
On PowerPC/POWER and ARM platforms, it partially handles implicit locking
arising from load-linked and store-conditional instruction pairs.  

### OpenMP 
OpenMP is a tool to parallelize already existing code. By using keywords like:
**pragma omp parrallel** we make a promise that the code bellow in is possible
to parallelize. 
