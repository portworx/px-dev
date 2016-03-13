#Walkthrough of Cassandra with PX-Lite
Apache Cassandra is an open source distributed database management system designed to handle large amounts of data across commodity servers. 

Setting up a Cassandra cluster with Portworx storage takes only a few commands. In this scenario we will create a three-node Cassandra cluster. 

In this example, we have three servers
* 10.0.0.1 is created in Step 1 and is the seed for Cassandra
* 10.0.0.2 is created in Step 3a
* 10.0.0.3 is created in Step 3b
 
When creating these servers in a public cloud, such as AWS, you can specify the each instance's private IP address in place of the 10.0.0.[1-3]. 
 
### Step 1: Create storage volumes for each instance
To create storage volumes for each instance, run the following command on each server. Note that the size=4 is specifying 4GB. 

```
    docker volume create -d pxd --opt name=cassandra_volume --opt \
    size=4 --opt block_size=64 --opt repl=1 --opt fs=ext4
```

The output of the command is the volume identifier, which we refer to ```DOCKER_CREATE_VOLUME_ID``` in the command-line examples in this guide. This volume identifier can be later retrived by running  ```docker volume ls ```. 


### Step 2: Start the Cassandra Docker image on node 1
We will use the docker -v option to assign the volume we created with docker volume create. Reminder: your DOCKER_CREATE_VOLUME_ID passed into the -v option can be retrievied by running ```docker volume ls```. You should also substitute your IP address for the 10.0.0.1 placeholder in the CASSANDRA_BROADCAST_ADDRESS parameter. 

>Important: if you are running an OS with SELinux enabled, a workaround to issue [20834](https://github.com/docker/docker/pull/20834) is to pass [security-opt] (https://github.com/portworx/px-lite/blob/master/faq.md) parameter between 'run' and '--name'.

```
    docker run --name cassandra1 -d \
    -p 7000:7000 -p 7001:7001 -p 9042:9042 -p 9160:9160 \
    -e CASSANDRA_BROADCAST_ADDRESS=10.0.0.1 \
    -v [DOCKER_CREATE_VOLUME_ID]:/var/lib/cassandra cassandra:latest
```

### Step 3: Start Docker on the other nodes 
Create a new volume for the Cassandra instance on node 2 by running the ``docker volume create ``, as shown in step 1. Afterwards, the only difference from the previous docker run command is the addition of the -e CASSANDRA_SEEDS=10.0.0.1 parameter. This is a pointer to the IP address of the first Cassandra node.  
  
#### Step 3a: On Cassandra node 2 run the following:
 ```
    docker run --name cassandra2 -d \
    -p 7000:7000 -p 7001:7001 -p 9042:9042 -p 9160:9160 \
    -e CASSANDRA_BROADCAST_ADDRESS=10.0.0.2 \
    -e CASSANDRA_SEEDS=10.0.0.1 \
    -v [DOCKER_CREATE_VOLUME_ID]:/var/lib/cassandra cassandra:latest
```

As in the above, create a new volume for the Cassandra instance on node 3 by running the ``docker volume create ``, shown in step 1.

#### Step 3b: On Cassandra node 3 run the following:
```
    docker run --name cassandra3 -d \
    -p 7000:7000 -p 7001:7001 -p 9042:9042 -p 9160:9160 \
    -e CASSANDRA_BROADCAST_ADDRESS=10.0.0.3 \
    -e CASSANDRA_SEEDS=10.0.0.1 \
    -v [DOCKER_CREATE_VOLUME_ID]:/var/lib/cassandra cassandra:latest
```
Remember to change the IP addresses in our examples to the ones used by your instances. It can take up to 30 seconds for Cassandra to start up on each node. To determine when your cluster is ready for use, view the logs: You should see messages that each node is part of the cluster.

Back to [PX-Lite](https://github.com/portworx/px-lite/).
