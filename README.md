# kafka-kraft-cluster-setup
Docker file and some instructions to setup Kafka 3.6 cluster without Zookeeper using Kraft protocol

#### Objective: The docker-compose.yaml is supposed to create a 3 node Kafka Cluster with 
- 1 node : controller, broker
- 2 nodes: broker

***Note: In this configuration the controller will not have any high availability.***

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
