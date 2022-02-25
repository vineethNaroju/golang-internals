# Un-buffered channels
- Receiver first => sender writes to receivers stack
- Sender first => receiver directly reads from sudog

# select case
- All channels are locked
- Sudog is put in sendq / recvq of all channels
- Channels unlocked and select-ing G is paused
- CAS operation, so one is winning
- resuming mirrors pause sequence

# Performance
- Calling into runtime scheduler, OS thread is unblocked
- Cross-gorountine stack reads and writes
- Go-routine wake-up path is lockless
- Potentially fewer memory copies

# CONS
- Memory management
- Garbage collection
- Stack shrinking