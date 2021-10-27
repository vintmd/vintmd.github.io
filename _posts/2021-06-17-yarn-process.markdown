# yarn process

![1623901318714](https://vintmd.github.io/photo/1623901318714.png)

1）client subbmit an application to ask the resource manager to get an instance of application master.

resource manager get back an application ID which used by client to watch resource infomation.

2)  resource manager find a useful container in node manager, then run the application master instance in this container.(app mstr run in one of the container, this also can seem as one container)

3) app master regist info into the RM

4)  app master send the resource request ask for RM to give container resource.

5)  when RM asign the container success, AM send launch message to node manager to start container.

6) job run in container, give the progress and status send to app master, then app master use the heart beat report to the RM, app master control whether release some parts of container.

7) client ask the status and progress to app master.

8) when finish the job, app master send cancel message to RM, all container given back to system.

RM tell node manager to merge the log and clean the file of relate containers