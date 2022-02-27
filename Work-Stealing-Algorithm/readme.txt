*** About me ***

Goroutines are not OS threads, and not green threads (threads managed by OS). They are higher level of abstraction of co-routines.
Co-routines are simply concurrent subroutines (functions / closures / methods in Go) that are non-preemptive but have multiple points
throughout to allow for suspension / re-entry.

M:N scheduler - M green threads onto N OS threads. Go-routines are scheduled onto M green threads. Go follows fork-join model.
Fork -> At any point in program, it can split off a child branch of execution to be run concurrently with its parent.
Join -> At some point in future, these concurrent branches of execution will join back together.
Join-Point -> The point where child rejoins parent.

*** Work-Stealing-Algorithm in a thread of execution ***

1. At a fork point, add tasks to the tail of deque associated with the thread.
2. If the thread is idle, steal work from the head of deque of some random thread.
3. At a join point that can't be solved yet, pop work off the tail of thread's own deque.
4. If the thread's deque is empty, then 
    a. Wait at join.
    b. Steal work from the head of a random thread's deque.

*** Example ***

package main

import "fmt"

func main() {
    fib := func(n int) <-chan int {
        results := make(chan int)
        go func() {

        }()
        return results
    }

    fmt.Println("fib(4) = %d", <-fib(4))
}

*** Flow ***

Say we have 2-core cpu, with T1 and T2 OS threads.

stack => top -> bottom
deque => front <-> back

---------------------------------------------------
T1-call-stack main
T1-work-deque
T2-call-stack
T2-work-deque

---------------------------------------------------
T1 pushes fib(4) to back of T1-deque
T1-call-stack main 
T1-work-deque fib(4)
T2-call-stack 
T2-work-deque

---------------------------------------------------
Say T1 wins the work steal, either of them can win but lets assume for now.
T1-call-stack fib(4), main(unsolved join)
T1-work-deque 
T2-call-stack
T2-work-deque

---------------------------------------------------
T1 pushes fib(3), fib(2) to the back of T1-deque.
T1-call-stack fib(4), main(unsolved join) 
T1-work-deque fib(3), fib(2)
T2-call-stack
T2-work-deque

---------------------------------------------------
T2 is idle and pops from front of T1-deque.
T1-call-stack fib(4), main(unsolved join)
T1-work-deque fib(2)
T2-call-stack fib(3)
T2-work-deque

---------------------------------------------------
T1 can't work due to fib(4) waiting on channel output of fib(3) and fib(2)
T1 pops off from back of T1-deque 
T1-call-stack fib(2), fib(4)(unsolved join), main(unsolved join)
T1-work-deque 
T2-call-stack fib(3)
T2-work-deque

---------------------------------------------------
T2 pushes fib(2) and fib(1) to back of T2-deque
T1-call-stack fib(2), fib(4)(unsolved join), main(unsolved join)
T1-work-deque
T2-call-stack fib(3)
T2-work-deque fib(2), fib(1)

---------------------------------------------------
T1 returns base case of fib(2)
T1-call-stack return 1, fib(4)(unsolved join), main(unsolved join)
T1-work-deque
T2-call-stack fib(3)
T2-work-deque fib(2), fib(1)

---------------------------------------------------
T2 pops work off of T2-deque
T1-call-stack return 1, fib(4)(unsolved join), main(unsolved join)
T1-work-deque
T2-call-stack fib(1), fib(3)(unsolved join)
T2-work-deque fib(2)

---------------------------------------------------
T1 is idle and so pops off work from front of T2-deque.
T1-call-stack fib(2), fib(4)(unsolved join), main(unsolved join)
T1-work-deque
T2-call-stack fib(1), fib(3)(unsolved join)
T2-work-deque

---------------------------------------------------
T2 returns base case of fib(1)
T1-call-stack fib(2), fib(4)(unsolved join), main(unsolved join)
T1-work-deque
T2-call-stack return 1, fib(3)(unsolved join)
T2-work-deque

---------------------------------------------------
T1 returns base case of fib(2) and fib(3) join is ready
T1-call-stack return 1, fib(4)(unsolved join), main(unsolved join)
T1-work-deque
T2-call-stack return 1, fib(3)(unsolved join)
T2-work-deque

---------------------------------------------------
We have solved join of fib(3) on T2-call-stack
T1-call-stack fib(4)(unsolved join), main(unsolved join)
T1-work-deque
T2-call-stack return 2
T2-work-deque

---------------------------------------------------
We have solved join of fib(4) on T1-call-stack.
T1-call-stack return 3, main(unsolved join)
T1-work-deque
T2-call-stack
T2-work-deque

---------------------------------------------------
We have solved join of fib(4) on T1-call-stack.
T1-call-stack print 3
T1-work-deque
T2-call-stack
T2-work-deque