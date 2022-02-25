# Un-contended case
Cost of atomic CAS

# Contended case
Worst case :
Failed atomic CAS + spinning + go-routine context switch + thread context switch
.......depends on degree of contention

? How does app performance change with concurrency ?
? How many thread do we need to support a target throughput, keeping same response time ?
? How does response time change with number of threads, with constant workload ?

# Amdahls law
speed-up with N thread => 1 / (1 - p + p/N)

Amdahls doesn't account for a co-ordination penalty

# Universal Scalablity Law (USL)
contention (due to serialization of shared resources)=> xN
- lock contention, database contention
cross-talk (due to co-ordination for coherence) => yN*N
- servers co-ordinating to sync mutable state

Throughput => N / (xN + yN*N + C)

So, in most parallel workloads throughput flattens much later than in serial workloads