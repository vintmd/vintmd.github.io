# Google file system review

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

![1623293837727](https://vintmd.github.io/photo/1623293837727.png)

1) master control the each chunk server, each block locations and other meta infomations

2) client ask the file chunk location to master, send data to random(nearly) chunkserver

3) chunkserver organize the each chunk replicas, replica the data from client(or other chunkserver) to next 

secondary chunkserver



## Consistency model

![1623296366858](https://vintmd.github.io/photo/1623296366858.png)

1) master control the global total order which used to apply mutation to one chunk

2) use the chunk version to detect each chunk mutation(stale replicas delete by GC)

* ofs use the central xlog id asgin give the mutation order, avoid the consistency issues on different machine.

3) master use the heartbeat to the chunkserver infomation(checksum) to detect stale chunk

## Interaction

### Lease and mutaion order

![1623297437281](https://vintmd.github.io/photo/1623297437281.png)

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

![1623309149425](https://vintmd.github.io/photo/1623309149425.png)

when master  receive the snapshot request, it release all the chunk server leases which can give the control that write to chunk back to master. when during this time ouccr the chunk write, it can copy the new chunk C` and replica this new chunk to other replicas.

* ofs use the Cow to cut the memory used. [Cow on btree][http://www.bzero.se/ldapd/btree.html]



## Master process

### Namespace and locking

1) namespace same to the ofs each memory engine to maintain the struct of tree dir.

2) add the lock each level. 

* ofs memory engine use the global lock.
* ofs db engine use txn share mode to lock the parent inode.

![1623311148274](https://vintmd.github.io/photo/1623311148274.png)

### Replica placement

maximize data reliability and availability, and maximize net-wor kbandwidth utilization. 

### Creation, Re-replication, Rebalancing 

![1623314173358](https://vintmd.github.io/photo/1623314173358.png)

1) creation relate the space utilization, write behand(batch way) and the replica chunk spead.

2) re-replication according the priority(the number of lose chunk) to replic the lose chunk.

3) rebalancing master period to do according the average disk space.

* rebalancing in ofs, the root server can asign the file system to each different meta server

  

## GC

distributed garbage collection knowledge

after all opration the master trigger the period scan the chunks from the heart beat whether difference to the memory dir tree to decide whether clean the chunks!

![1623317050747](https://vintmd.github.io/photo/1623317050747.png)

* ofs gc and life cycle are all bypass system which offer the interface to operation db data

1) simple and reliable

2) merge storage reclamation into background

3) delay in reclaiming provide safety net against irreversible deletion

* which same to the ofs this way might influence the efficient



## Fault tolerance

### Available

1) fast recover

* ofs use the parallel load snapshot and apply xlog to fast recover

2) chunk replication

* same to data stream in cos which use parity or eraseure codes

3) master replication

record log replica between each master, shadow master provide the read-only access even master is done.

* like ofs use the election which follower catch up the xlog to keep the same dir tree to the master as far as possible.
* only one master can create and delete replicas to prevent the split of brain issues.



### Data intergrity

chunkserver period detect the stale chunk(each chunk broken up into 64KB blocks, each has 32 bit checksum)

* first and last blocks of the range being overwriten which need to be checked



### Diagnostic tools

logs and online monitor



## Performance mainest

1) read, write, append records

* ofs read and write give to the cos
* ofs append records give to the db
* write behand and read ahead 

2) google file system's read performance is better to write(write three or more replicas)



## Reference







