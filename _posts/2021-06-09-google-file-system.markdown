# goose file system

1) component failures

2) files huge

3) most files mutated by appending new data rather than overwriting

4) API



## Design

1) detect, tolerate, recover

2) modest number of large files

3) large streaming reads and small random reads

4) sequential writes that append data to files

5) concurrent append to the same file

6) data opration high sustained bandwidth(nearly)



## Arch

![1623293837727](C:\Users\alantong\Desktop\vintmd.github.io\photo\1623293837727.png)

1) master control the each chunk server, each block locations and other meta infomations

2) client ask the file chunk location to master, send data to random(nearly) chunkserver

3) chunkserver organize the each chunk replicas, replica the data from client(or other chunkserver) to next 

secondary chunkserver



## Consistency model

![1623296366858](C:\Users\alantong\Desktop\vintmd.github.io\photo\1623296366858.png)

1) master control the global total order which used to apply mutation to one chunk

2) use the chunk version to detect each chunk mutation(stale replicas delete by GC)

* ofs use the central xlog id asgin give the mutation order, avoid the consistency issues on different machine.

3) master use the heartbeat to the chunkserver infomation(checksum) to detect stale chunk

## Interaction

### Lease and mutaion order

![1623297437281](C:\Users\alantong\Desktop\vintmd.github.io\photo\1623297437281.png)

1) client ask the primary(hold the lease) and other chunk location

2) cache the master return info

3) replica data to replica chunkserver one by one(random)

4)  after replica all data, client ask the primary to decide the log order

* it is same to the ofs which insert the xlog id after commit meta data into db
* it decide the mutaion order by primary replica.

5) primary give the order to other replicas

6) return the info whether sussess  decide the mutaion order

7) give the result return to the client.

* lease give the control of master to control each chunk server!



## Automic record append

1) merge the scattered appended record.

2) replica the record append to each one chunkserver

* ofs use the txn to ensure the automic of the commit and xlog



## Snapshot

![1623309149425](C:\Users\alantong\Desktop\vintmd.github.io\photo\1623309149425.png)

when master  receive the snapshot request, it release all the chunk server leases which can give the control that write to chunk back to master. when during this time ouccr the chunk write, it can copy the new chunk C` and replica this new chunk to other replicas.

* ofs use the Cow to cut the memory used. [Cow on btree][http://www.bzero.se/ldapd/btree.html]



## Master process

### Namespace and locking

1) namespace same to the ofs each memory engine to maintain the struct of tree dir.

2) add the lock each level. 

* ofs memory engine use the global lock.
* ofs db engine use txn share mode to lock the parent inode.

![1623311148274](C:\Users\alantong\Desktop\vintmd.github.io\photo\1623311148274.png)

### Replica placement

maximize data reliability and availability, and maximize net-wor kbandwidth utilization. 

### Creation, Re-replication, Rebalancing 

![1623314173358](C:\Users\alantong\Desktop\vintmd.github.io\photo\1623314173358.png)

1) creation relate the space utilization, write behand(batch way) and the replica chunk spead.

2) re-replication according the priority(the number of lose chunk) to replic the lose chunk.

3) rebalancing master period to do according the average disk space.

* rebalancing in ofs, the root server can asign the file system to each different meta server

  

## GC

distributed garbage collection knowledge

after all opration the master trigger the period scan the chunks from the heart beat whether difference to the memory dir tree to decide whether clean the chunks!

![1623317050747](C:\Users\alantong\Desktop\vintmd.github.io\photo\1623317050747.png)







## Pre

this is the sub test





