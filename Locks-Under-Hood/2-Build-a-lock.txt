# Mutual Exclusion

var tasks Tasks // shared ring buffer

T1 running on CPU-1
func Reader() {
    t := tasks.Get()
    // do something with it
}

T2 running on CPU-2
func Writer() {
    tasks.Put(t)
}

------------------------
Using a flag

// access(0) or not (1)
var flag int
var tasks Tasks


T1
func Reader() {
    for {
        if flag == 0{
            flag++
            t := tasks.Get()
            // do something
            flag--
            return
        }
    }
}

T2
func Writer() {
    for {
        if flag == 0{
            flag++
            // do something
            flag--
            return
        }
    }
}

flag++ => in assembly as INCQ instruction => CPU => 

1. Read(0) from Memory 
2. Modify in CPU 
3. Write(1) to  Memory

T2's RMW maybe interleaved with T1's RMW ----> Non-Atomic operation

# Atomicity

In x86_64 loads, stores that are naturally aligned upto 64b
It gaurantees that data item fits within a cache line - a consistent view across all cores
for a single cache line, called as Cache coherence.

# Memory access Re-order Operations by Compiler and Processor

flag = 1
t := tasks.Get()
flag = 0

Store load reordering - load t before store flag = 1
Writing is expensive because of cache coherency, so we read while write happens.

Rule - Sequential consistency for single threaded programs
C++, Go guarantee that data-race free programs will be sequentially consistent.

x86_64 provides Total Store Ordering (TSO) - a relaxed consistency model
most reorderings are invalid but store load is fine - this allows processor to hide write latency