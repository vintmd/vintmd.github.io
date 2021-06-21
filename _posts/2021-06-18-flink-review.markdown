# Flink review

Apache Flink is a framework and distributed processing engine for stateful computations over *unbounded and bounded* data streams.  

suit ETL, analytics and event-driven applications.



## Process of commit job

flink can use the yarn, apache mesos and k8s to manage the resource.

as the photo shows the yarn commit job process (mesos and k8s on the other article learn):

1) Flink client send the jars and configuartions to hdfs.

2) Flink client submit the job to the resource manager.

3) Resource manager start the application master in one node manager and init the job manager

4) application master apply the resource to 5) start task manager on each node manger

![1624016394384](https://vintmd.github.io/photo/1624016394384.png)



## Anatomy of a flink cluster

![image-20210621145910650](https://vintmd.github.io/photo/image-20210621145910650.png)

1) flink runtimes consists of two types of processes: a job manager and one or more task managers

2) client is used to prepare and send a dataflow to the job manager

3) job manager contains the a) resource manager which manages the task slots, which are the unitof resource sheduling in flink cluster; b) dispatch provides a rest interface to submit flink applications for execution and starts a new job master for each submit job; c) job master manage the execution of single job grahp, multiple jobs can run simultaneously each having its own job master.

4) task manager execute the tasks of a dataflow, and buffer and exchange the data streams



## Tasks and Operator Chains

![image-20210621151444846](https://vintmd.github.io/photo/image-20210621151444846.png)

flink chains operator subtasks together into tasks.

chains operators togehter into tasks is a useful optimization: 

it reduces the overhead of thread to thread handover and buffering.

and increase overall throughput while decreasing latency. 



## Task Slots and Resources

1) flink cluster needs exactly as many task slots as the highest parallelism used in the job

2) It is easier to get better resource utilization. without slot sharing, the non-intensive source/map() subtasks woulod block as many resources as the resource intensive window subtask.



## Exactly once End-to-end

1) your sources must be replayable and

2) your sinks must be transactional( or idempotent)



## S3 sink connector

connector sink the data to file system which implement the recoverable relate interface.

![image-20210621153323615](https://vintmd.github.io/photo/image-20210621153323615.png)

file in each bucket contains three status: pending files, in-progress file and finished file.

the mainest process of flink recover is use the checkpoint records the file path and offset, then truncate the in-progress file to relate position. and normal time append data to the in-progress file.

![image-20210621164857677](https://vintmd.github.io/photo/image-20210621164857677.png)

object storage not support the truncate and append operation, so use the MPU to replace the append and truncate. checkpoint record file upload id and each upload parts meta infomation. if not write done a whole part use the put object operation upload this single file as the un-finish part which can be download continue to append write in local memory.



![image-20210621153154271](https://vintmd.github.io/photo/image-20210621153154271.png)

![image-20210621164223236](https://vintmd.github.io/photo/image-20210621164223236.png)

![image-20210621164325500](https://vintmd.github.io/photo/image-20210621164325500.png)

what a magical thing MPU!



## Reference

[1] https://flink.apache.org/flink-architecture.html

[2] https://zhuanlan.zhihu.com/p/337222474

[3] https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/learn-flink/fault_tolerance/









