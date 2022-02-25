
# Go mutex contention profiler
perf-lock
Dtrace / systemtap
mutrace
valgrind-drd

# Don't use lock
- remove sync from hot paths
- use atomic ops
- use lock free data structures => per-core runqueues

# Use granular locks
- Shard data (ensure false sharing)
- per-processor locks (linux scheudler per-cpu run-queues)

# Less serial work
- use mutex profiler