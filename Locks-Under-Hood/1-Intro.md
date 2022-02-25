# Intro

Go runtime uses locks extensively.

# Case Study

- x86_64 SMP machine running Linux.
- All cores share a single memory bus

(Go routines are handled by go runtime not by os)

# In brief

Data shared between go-routines must be synchronized.
One is to use blocking, non-recursive lock as,

var mutex sync.Mutex
mutex.Lock()
critical-section
mutex.Unlock()