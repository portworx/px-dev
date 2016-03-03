#Walkthrough of Cassandra with PX-Lite
Apache Cassandra is an open source distributed database management system designed to handle large amounts of data across commodity servers. 

Setting up a Cassandra cluster with Portworx storage takes only a few commands. In this scenario we will create a three-node Cassandra cluster. 
 
### Step 1: Create storage volumes for each instance
To create storage volumes for each instance, run the following command on each server. Note that the size=4 is specifying 4GB. 

```
    docker volume create -d pxd --opt name=cassandra_volume --opt \
    size=4 --opt block_size=64 --opt repl=1 --opt fs=ext4
```

The output of the command is the volume identifier. Store this for later; you will need the identifier to start the containers.


Before you start the Docker containers, you also need to tell Cassandra what IP address to advertise to other Cassandra nodes. In this scenario we will use 10.0.0.1 as the IP address for the first Cassandra node; 10.0.0.2, for the second Cassandra node, and 10.0.0.3, for the third Cassandra node.

### Step 2: Start the Cassandra Docker image on node 1
We will use the docker -v option to assign the volume we created with docker volume create. Substitute the DOCKER_CREATE_VOLUME_ID for the volume ID that was returned from docker volume create. You should also substitute your IP address for the 10.0.0.1 placeholder in the CASSANDRA_BROADCAST_ADDRESS parameter. 

```
    docker run --name cassandra1 -p 7000:7000 -p 9042:9042 -p \ 9160:9160 -e CASSANDRA_BROADCAST_ADDRESS=10.0.0.1 \
    -v DOCKER_CREATE_VOLUME_ID:/var/lib/cassandra cassandra:latest
```

### Step 3: Start Docker on the other nodes 
The only difference from the previous docker run command is the addition of the -e CASSANDRA_SEEDS=10.0.0.1 parameter. This is a pointer to the IP address of the first Cassandra node.  
  
 On Cassandra node 2 run the following:
 ```
    docker run --name cassandra2 -d -p 7000:7000 -p 9042:9042 \ 
    -p 9160:9160 -e CASSANDRA_BROADCAST_ADDRESS=10.0.0.2 \
    -e CASSANDRA_SEEDS=10.0.0.1 \
    -v DOCKER_CREATE_VOLUME_ID:/var/lib/cassandra cassandra:latest
```
On Cassandra node 3 run the following:
```
    docker run --name cassandra3 -d -p 7000:7000 -p 9042:9042 \ 
    -p 9160:9160 -e CASSANDRA_BROADCAST_ADDRESS=10.0.0.3 \
    -e CASSANDRA_SEEDS=10.0.0.1 \
    -v DOCKER_CREATE_VOLUME_ID:/var/lib/cassandra cassandra:latest
```
Remember to change the IP addresses in our examples to the ones used by your instances. It can take up to 30 seconds for Cassandra to start up on each node. To determine when your cluster is ready for use, view the logs: You should see messages that each node is part of the cluster.
