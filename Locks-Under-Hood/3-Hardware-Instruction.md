# Special hardware instructions

For guaranteed atomicity
ex: XCHG in x86
To prevent memory reordering - Memory barriers - prevent compiler reordering as well
ex: MFENCE, LFENCE, SFENCE

For both of them we have x86 LOCK instruction prefix, atomic operation like atomic.Add in Golang
LOCK ADD -> atomic.Add
LOCK CMPXCHG -> atomic.CompareAndSwap (CAS)

CAS conditionally updates a variable, checks if it has expected value and if so, changes it to desired value.

First try -

var flag int
var tasks Tasks

func Reader() {
    for {
        if atomic.CompareAndSwap(&flag, 0, 1) {
            // get tasks

            atomic.Store(&flag, 0)
            return
        }
    }
}

The above is called as Spinlock - used extensively in linux kernel.

!!! Spinning for long duration is wasteful - takes away CPU time from other threads

# Linux's Futex

Interface(futex syscall) and mechanism for userspace code to ask kernel to suspend / resume threads.

var flag int (0 - unlocked, 1 - locked, 2 - there's a waiter)
var tasks Tasks

func Reader() {
    for {
        if atomic.CompareAndSwap(&flag, 0, 1) {
            ....
        }

        // CAS failed, set flag to sleeping
        v := atomic.Xchg(&flag, 2)

        futex(&flag, FUTEX_WAIT, ...) // we tell kernel to suspend us until flag changes
    }
}


In kernel - 
1. Arrange for thread to be resumed in future
    - Hash the user space address of flag and store the entry in kernel queue (futex_q)
2. Deschedule the calling thread to suspend it

func Writer() {
    for {
        if atomic.CompareAndSwap(&flag, 0, 1) {
            v := atomic.Xchg(&flag, 0)

            if v == 2 {
                futex(&flag, FUTEX_WAKE, ...)
            }

            return
        }

        v := atomic.Xchg(&flag, 2)
        futex(&flag, FUTEX_WAIT, ...)
    }
}

pthread mutexes use futexes

# Cost of a Futex
1-thread CAS = 13ns
12-thread CAS + syscall + thread context switch = 0.9us

Go futex -> spin sometime, if fail then make futex syscall

# ... can we do better for user-space threads ?
Go-routines are user-space threads
- Go runtime multiplexes them onto threads
- Light weight and cheaper than threads:
    goroutine switches - tens of ns
    thread switches - a us

=> We can block goroutine without blocking underlying thread.

# Go runtime's Semaphore
- similar to futex but used to sleep / wake goroutines.
- Go-routine wait queues are managed by the runtime in user-space.

var flag int
var tasks Tasks

G1
func Reader() {
    for {
        if atomic.CompareAndSwap(&flag, ...) {

        }
        // CAS failed, add G1 as a waiter for flag
        root.Queue()
        // wait queues are managed by go runtime in user space

        // and suspend G1
        gopark()
        // Go-runtime deschedules G1 and thread keeps running
    }
}

bucket[hash(&flag)] = Treap ( {&flag, G1} -> {&other, [G3, G4]} )

G2
func Writer() {
    for {
        if atomic.CompareAndSwap(&flag, 0, 1) {

            // set flag to unlocked
            atomic.Xadd(&flag, ...)

            // find first waiter and reshedule it
            waiter := root.dequeue(&flag)
            goready(waiter)
            return
        }

        root.queue()
        gopark()
    }
}
