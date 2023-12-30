# kafka-kraft-cluster-setup
Docker file and some instructions to setup Kafka 3.6 cluster without Zookeeper using Kraft protocol

#### Objective: The docker-compose.yaml is supposed to create a 3 node Kafka Cluster with 
- 1 node : controller, broker
- 2 nodes: broker

> Note: In this configuration the controller will not have any high availability.

Directories needed for the docker-compose.yaml can be created using the makedir file:
The tree structure of the directories created would look something like this:
```
data
├── kafka-0
│   └── logs
├── kafka-1
│   └── logs
└── kafka-2
    └── logs
```

This docker-compose creates 3 containers : kafka-0, kafka-1 and kafka-2
Their bootstrap address is: kafka-0:19092,kafka-1:29092,kafka-2:39092

To be able to access the cluster, you will need to add these to your /etc/hosts files on Mac/Linux.

Sample:
```
192.168.68.111 kafka-0 kafka-1 kafka-2
```
> Note: <br>
> - In Mac/Linux, above entry can be added in /etc/hosts, the first column is the Machine's IP where the kafka containers are running.
> - In Windows, the hosts file can be found at: c:\Windows\System32\Drivers\etc\hosts

<br><hr><br>
#### Post setup

For running the following commands, you have two choices:
- Download Kafka binaries from ```https://kafka.apache.org/downloads```, and run the commands from bin directory. (You will need Java to be already setup)
- You can run the commands from inside one of the 3 containers.


###### Check Kafka setup:
```kafka-metadata-quorum.sh --bootstrap-server kafka-0:19092 describe --status```
<br>
output should look something like this:
```
ClusterId:              TkU3oEVVNTZwNJJENDL2Q0
LeaderId:               0
LeaderEpoch:            2
HighWatermark:          4586
MaxFollowerLag:         0
MaxFollowerLagTimeMs:   0
CurrentVoters:          [0]
CurrentObservers:       [1,2]
```
Above output shows that there is 1 controller and 2 other brokers, it also shows Node:0 is the leader.


