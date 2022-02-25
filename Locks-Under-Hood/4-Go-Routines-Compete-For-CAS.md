# Resumed go-routines have to compete with other routines for CAS

It's possible they will likely fail to acquire CAS, since there is delay between flag being unlocked
and rescheduling a waiter from queue.

- Unnecessarily resuming a waiter go-routine (context switch again ... aaaaaaah)
- cause go-routine starvation (long wait times, high tail latencies)

~ add more code and we get sync.Mutex TADAAAAA !!!!!!!!

# sync.Mutex - Hybrid lock - spinlock for sometime and then use semaphore to sleep.
- tracks state to prevent resuming a waiter go-routine
    - an unlock doesn't wake up a waiter
- prevent starvation
    - waiter loses CAS, it's scheduled at queue head.
    - waiter fails to lock for 1ms, mutex -> starvatiom mode
        - other go-routines must queue, they can't CAS
        - unlock hands the mutex to first waiter i.e no competition.

# Performance
1 go-routine = 13ns
12 go-routines = 0.8us

# Go vs C
- Go performs better than C at low concurrency but they converge at high concurrency.
- sync.Mutex -> semaphore -> hash table bucket needs a lock - is FUTEXXXXX !!!!
- thread contention at semaphore bucket locking.
- Futexes -> SPIN-LOCKSSSSS !!!!!!!!!!!