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







## Reference

[1] https://flink.apache.org/flink-architecture.html

[2] https://zhuanlan.zhihu.com/p/337222474

[3] https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/learn-flink/fault_tolerance/