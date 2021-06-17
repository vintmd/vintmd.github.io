#  MR

data processing on large clusters

1) process a kv par to generate a set of intermediate kv pairs

2) reduce merge all intermediate values with same key



## Programming model

### Implementation

1) user program fork the worker and master

2) master assign the map and reduce to the worker

3) map worker read each line of input files and it to local 

4) reduce worker remote read the intermediate files

![1623813146557](https://vintmd.github.io/photo/1623813146557.png)



### Master data structures

worker's task(map or reduce) has three status idle, in-progress and completed, master communcate to 

workers to get status and record the intermediate files(local disks)

![1623827057310](https://vintmd.github.io/photo/1623827057310.png)

### Fault Tolerance

#### worker failure

master ping every worker find the no-response worker

1)  ping failed which no-response

2)  map task set back to idle state and eligible for rescheduling

* map task's intermediate file is on local disk which need to re-scheudle
* reduce task's output file is global file system which no need to re-scheudle

3)  master broadcast to each worker let the reducer read data from normal worker

![1623828957875](https://vintmd.github.io/photo/1623828957875.png)

#### master failure

write periodic checkpoints of the master data structures. use the checkpoint to recover.



#### task granularity

1)  pieces of maps and reduces should be much larger than the number of worker

* improve the load balance 
* speed up the recovery when workers fail

2) pieces of reduce decided by user which define the number of output files.

![1623843856874](https://vintmd.github.io/photo/1623843856874.png)



## Refinements

### Partitioning Function

1) hash(key) mod R

2) or according to some specific case hash function



### Ordering Guarantees

guarantee within a given partition. which the intermediate kv pairs are processed in increasing key order.



### Combiner Function

Combine function is executed on each machine that performs a map task that does partial merging of this data before it is sent over the network



### Side-effects

application writes to a temp file and atomically renames this file once it has been fully generated.

map reduce not provide support for atomic two-phase commits of mulitple output files produced by a single task.

* ask self why there are need the tempory file then rename it, what problem can solve by this.



### Skipping bad records

there may be some bugs to make behavier wrong,  each worker process installs a signal handler that 

catches segmentation violations and bus errors.  when master has seen more than one failure on a particular record, it skip it.



### Loal Execution

easy for local debug and test.



### Status information

master collect the informations of status of each workers and exports a set of status pages.



##  Performance

grep, sort, effect of backup tasks, machine failures.

the bandwidth cost between the map task and reduce task.

map cost > shuffle cost > reduce cost (GFS replciate cost > EC cost)