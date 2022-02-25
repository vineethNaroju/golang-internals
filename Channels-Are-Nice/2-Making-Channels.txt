
# Make chan

buffch := make(chan Task, 3)
unbuffch := make(chan Task)

# hchan struct
- buf -> circular queue / ring buffer
- sendx -> send index
- recvx -> receive index
- mutex -> lock

* make(chan Task) returns a pointer to allocated struct on heap space.

# Demo 

G1
func main() {
    ...
    for _, task := ranhe tasks {
        taskCh <- task
    }
}

G2
func worker(taskChan <-chan Task) {
    for {
        task := <-taskChan
        process(task)
    }
}

# Don't communicate by sharing memory, share memory by communicating.
~ We just share hchan struct but we make copies of Task from buffer

# Say channel is full, G1 is paused and resumed after a receive.

Run-time scheduler RS
go-routines are simply user space threads (not managed by OS)
RS schedules go-routines onto OS threads.

# GO's M:N scheduling
M - OS thread
G - goroutine
P - context for scheduling

Ps hold run-queues
In order to run G, M must hold a P

M (currentG = G-running, P-runQ[G1, G2] runnable ones)

# Say G1 comes before G2
# G1 is blocked on send over channel 
G1
ch <- task4
gopark -> calls into scheduler
sets G1 to waiting
removes G1 from M

# Resuming go-routines

type hchan struct {
    sendq of sudog (waiting senders)
    recvq (waiting receivers)
}

type sudog struct {
    G - waiting Go-routine
    elem - to send / recv
}
--------------------------
G1
ch <- task4
sendq.push(sudog{G, elem(task4)})

G2
t := <-ch
pop off task1
pop off sudog of G1 and push to buffer
goready(G1) -> calls scheduler -> sets G1 to runnable -> run-queue
returns to G2


# Say G2 comes before G1

G2
t := <-ch
set up state for resumption (put sudoq in recq) and gopark(G2).
recvq[sudog{G2, elem(t)}]

we could - enqueue task and goready(G2)

or 

'G1 writes to t directly of G2'
G1 writes to G2's stack
G2 doesn't need to acquire to lock and manipulate buffer

goready(G2)
