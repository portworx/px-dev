![logo](http://i.imgur.com/l8JRhxg.jpg)

# PX-Lite alpha

PX-Lite aggregates the storage capacity of hard drives on your server and it clusters multiple servers for high availability. As you develop and deploy your apps in containers, use PX-Lite for elastic storage capacity, managed performance, and high availability.

### See our quick-start guides on running PX-Lite for scenarios such as:
-	Hosting WordPress with persistent storage
-	Scaling a Cassandra database
-	Running the Docker registry with high availability

Use our command-line tools to directly manage volumes, such as snapshotting a container’s storage.  We will add docs to our RESTful interface soon.

*Stay tuned for updates on PX-Lite and our PX-Enterprise release.*



## Architecture and Storage
Portworx storage is deployed as a container and runs on a cluster of servers. Application containers provision storage directly through the Docker [*volume plugins*](https://docs.docker.com/engine/extend/plugins_volume/#command-line-changes:be52bcf493d28afffae069f235814e9f)  API or the Docker [*command-line*](https://docs.docker.com/engine/reference/commandline/volume_create/). 

Administrators and DevOps can alternatively pre-provision storage through the Portworx command-line tool (`pxctl`) and then set storage policies using the Portworx administrative interface.  Using `pxctl`, administrators can set container granular snapshot policies, create clones of volumes and set CoS parameters.

### Portworx storage runs in a cluster of server nodes.
-   Each server has the PX-Lite container and the Docker daemon.
-   Servers join a cluster and share config through the key/value store, such as etcd.
-   The PX-Lite container pools the capacity of the storage media residing on the server.  You easily select storage media through the config.json file.

See [*Deployment Requirements*](#Requirements-and-Limitations) for compatibility requirements.

![](http://i.imgur.com/OWOedkS.png)

Storage volumes are thinly provisioned, using capacity only as an application consumes it.  Volumes are replicated across the nodes within the cluster, per a volume’s configuration, to ensure high availability.

Using MySQL as an example, a PX-Lite storage cluster has the following characteristics:

-   MySQL is unchanged and continues to write its data to /var/lib/mysql.
-   This data gets stored in the container’s volume, managed by PX-Lite.
-   PX-Lite synchronously replicates writes to the volume across the cluster.

Each volume specifies its request of resources (such as its max capacity and IOPS) and its individual requirements (such as ext4 as the file system and block size).

Using IOPS as an example, a team can chose to set the MySQL container to have a higher IOPS than an offline batch processing container. Thus, a container scheduler can move containers, without losing storage and while protecting the user experience.



## Installation and Tutorials
The following guides walk through setting up and using PX-Lite and maintaining storage. See the Deployment Requirements for details on the configuration.

### Step 1: Download and install the PX Kernel Module

PX-Lite runs as a Docker container, available on the DockerHub. This initial version of PX-Lite has a dependency on the [*lightweight*](http://github.com/portworx/px-fuse) kernel module, which must be installed on hosts.

You can download and install pre-built packages for select Centos and Ubuntu Linux distributions. If your kernel version is not listed in the table below, you can build the kernel module by following the instructions here: http://github.com/portworx/px-fuse

Find out your kernel version. For example:

```
# uname -r
3.19.3-1.el7.elrepo.x86_64
```

Install the kernel module on hosts using a command below.

| **Distribution**   | **Kernel** | **Download URL and installation command**                                                                                                                                            |
|  ----------------- | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Centos 7.0         | 3.10       | [*http://get.portworx.com/builds/Linux/centos/7/px-3.10.0-229.14.1.el7.x86\_64.rpm*](http://get.portworx.com/builds/Linux/centos/7/px-3.10.0-229.14.1.el7.x86_64.rpm) rpm -ivh px-3.10.0-229.14.1.el7.x86\_64.rpm |
| Centos 7.0         | 3.19.3     | [*http://get.portworx.com/builds/Linux/centos/7-3.19.3/px-3.19.3-1.el7.elrepo.x86\_64.rpm*](http://get.portworx.com/builds/Linux/centos/7-3.19.3/px-3.19.3-1.el7.elrepo.x86_64.rpm) rpm -ivh px-3.19.3-1.el7.elrepo.x86\_64.rpm |
| Ubuntu 14.04       | 3.13       | [*http://get.portworx.com/builds/Linux/ubuntu/14.04/px\_3.13.0-74\_amd64.deb*](http://get.portworx.com/builds/Linux/ubuntu/14.04/px_3.13.0-74_amd64.deb)                     dpkg --install px\_3.13.0-74\_amd64.deb |
| Ubuntu 15.04       | 3.19       | [*http://get.portworx.com/builds/Linux/ubuntu/15.04/px\_3.19.0-43\_amd64.deb*](http://get.portworx.com/builds/Linux/ubuntu/15.04/px_3.19.0-43_amd64.deb) dpkg --install px\_3.19.0-43\_amd64.deb |
                                  
On `centos` for example, this module can be installed the following way:

```
# wget http://wilkins.portworx.com:8080/job/PX-FUSE/lastSuccessfulBuild/artifact/go/src/github.com/portworx/px-fuse/rpm/out/px-3.19.3-1.el7.elrepo.15.x86_64.rpm
# rpm -ivh px-3.19.3-1.el7.elrepo.15.x86_64.rpm
```

### Step 2: Install Docker

PX-Lite requires [*Docker*](https://docs.docker.com/engine/installation/) version 1.10 or later. If you are using systemd, make sure that the Docker daemon is configured to let PX-Lite export storage.

### Step 3: Edit the JSON configuration

The PX-Lite `config.json` specifies the key-value store for the cluster and lets you select which storage devices PX-Lite will use.

To download the sample config.json file:
https://github.com/portworx/px-lite/blob/master/conf/config.json

Create a directory for the configuration file and move the file to that directory. This directory later gets passed in on the Docker command line.

```
# wget http://get.portworx.com/px-lite/config.json
# sudo mkdir -p /etc/pwx
# sudo cp -p config.json /etc/pwx
```

Here is a sample config.json file.
```
{
  "clusterid": "5ac2ed6f-7e4e-4e1d-8e8c-3a6df1fb61a5",
  "kvdb": "http://etcd.example.com:4001",
  "storage": {
   "devices": [
    "/dev/xvdf",
    "/dev/xvdg"
   ]
  }
}
```  
  
In the configuration file, make the `clusterid` unique among clusters in your key-value store.  Point the `devices` to local unused block device on your system, for example /dev/sdb. 

      Warning!!!: Any storage device that PX-Lite uses will be reformatted.

|  Field     |  Description                                                                                                    |  Example                          | Required
|  ---------- ---------------------------------------------------------------------------------------------------------------- | --------------------------------- | ----------
| clusterid  | A unique identifier for your cluster. Be sure to use the same identifier on all the nodes you want to cluster.  | 5ac2ed6f-7e4e-4e1d-8e8c-3a6df1fb61a5                 | required
| mgtiface   | The network interface for management data.                                                                      | eth0                              | optional
| dataiface  | The network interface for data transfers.                                                                       | eth1                              | optional
| kvdb       | The URI to your etcd server.                                                                                    | https://myetcd.example.com:4001   | required
| required
| devices    | The list of devices that PX-Lite will use. Any disks listed will be reformatted for PX use.                     | /dev/xvda                         | required


To find local drives that are available for use on your system, you can issue this bash command:
```
#!/bin/bash
for d in `find /dev/disk/by-uuid/ -type l | \
   xargs readlink -f | \
   awk '{print $1}'`; do
   is_mounted=0
   for m in `find /dev/disk/by-uuid/ -type l | \
       xargs readlink -f | \
       xargs -i grep {} /proc/mounts | \
       awk '{print $1}'`; do

       if [ $d == $m ]; then
           # echo "    Disk is mounted: $d"
           is_mounted=1
           break
       fi
       #echo "     $m"
   done
   if [ $is_mounted -eq 0 ]; then
       echo "Disk is NOT mounted: $d"
   fi
   #echo $d
done
```

*Warning: Please ensure that disks are empty, to avoid data loss!*                                                                                                                                                                 
 
### Step 4: Running PX-Lite

Start the PX-Lite container with the following run command:
```
# sudo docker run --restart=always --name px-lite -d --net=host --privileged=true \
                 -v /run/docker/plugins:/run/docker/plugins                       \
                 -v /var/lib/osd:/var/lib/osd:shared                              \
                 -v /dev:/dev                                                     \
                 -v /etc/pwx:/etc/pwx                                             \
                 -v /opt/pwx/bin:/export_bin:shared                               \
                 -v /var/run/docker.sock:/var/run/docker.sock                     \
                 -v /var/cores:/var/cores                                         \
                 --ipc=host                                                       \
                portworx/px-lite
```

     
##### Explanation of the runtime command options:

    --privileged
        > Sets PX-Lite to be a privileged container. Required to export block  device and for other functions.
        
    --net=host
        > Sets communication to be on the host IP address over ports 9001 -9003. Future versions will support separate IP addressing for PX-Lite.
        
    --shm-size=384M
        > PX-Lite advertises support for asynchronous I/O. It uses shared memory to sync across process restarts
        
    -v /run/docker/plugins
        > Specifies that the volume driver interface is enabled.
        
    -v /dev
        > Specifies which host drives PX-Lite can see. Note that PX-Lite only uses drives specified in config.json. This volume flage is an alternate to --device=\[\].
        
    -v /etc/pwx/config.json:/etc/pwx/config.json
        > the configuration file location.
        
    -v /var/run/docker.sock
        > Used by Docker to export volume container mappings.
        
    -v /var/lib/osd:/var/lib/osd:shared
        > Location of the exported container mounts. This must be a shared mount.
        
    -v /opt/pwx/bin:/export_bin:shared
        > Exports the PX command line (pxctl) tool from the container to the host.



## Testing

### Creating a volume with Docker:

Refer to  [https://docs.docker.com/engine/reference/commandline/volume_create/](https://docs.docker.com/engine/reference/commandline/volume_create/)

```
# docker volume create -d pxd --name <volume_name>
```
Everything else is optional. Use --opt to specify optional parameters; use the same option keywords as pxctl.

For Example: 

```
  docker volume create -d pxd --name <volume_name> --opt fs=ext4 --opt size=10G
```

### Creating a volume with pxctl:
```
# /opt/pwx/bin/pxctl volume create foobar
3903386035533561360
```

```
# /opt/pwx/bin/pxctl create volume --help
NAME:
   create volume - Create a volume

USAGE:
   command create volume [command options] [arguments...]

OPTIONS:
   --label, -l                  Comma separated name=value pairs, e.g name=sqlvolume,type=production
   --size, -s "1000"            specify size in MB
   --fs "ext4"                  filesystem to be laid out: none|xfs|ext4
   --seed                       optional data that the volume should be seeded with
   --block_size, -b "32"        block size in Kbytes
   --repl, -r "3"               replication factor [1..3]
   --cos "1"                    Class of Service: [1..9]
   --snap_interval, --si "0"    snapshot interval in minutes, 0 disables snaps

```

### Walkthrough of Cassandra with PX-Lite
Apache Cassandra is an open source distributed database management system designed to handle large amounts of data across commodity
servers.

Setting up a Cassandra cluster with Portworx storage takes only a few commands. In this scenario we will create a three-node Cassandra
cluster.

#### Step 1: Create storage volumes for each instance
To create storage volumes for each instance, run the following command on each server.


```
# docker volume create -d pxd --opt name=cassandra_volume --opt size=20000 --opt bs=64 --opt ha=1 --opt fs=ext4
```

The output of the command is the volume identifier. Store this for later; you will need the identifier to start the containers.

Before you start the Docker containers, you also need to tell Cassandra what IP address to advertise to other Cassandra nodes. In this scenario we will use 10.0.0.1 as the IP address for the first Cassandra node;
10.0.0.2, for the second Cassandra node, and 10.0.0.3, for the third Cassandra node.

#### Step 2: Start the Cassandra Docker image on node 1
We will use the docker -v option to assign the volume we created with docker volume create. Substitute the DOCKER\_CREATE\_VOLUME\_ID for the volume ID that was returned from docker volume create. You should also substitute your IP address for the 10.0.0.1 placeholder in the CASSANDRA_BROADCAST_ADDRESS parameter.

```
# docker run --name cassandra1 -p 7000:7000 -p 9042:9042 -p  9160:9160 -e CASSANDRA_BROADCAST_ADDRESS=10.0.0.1 -v DOCKER_CREATE_VOLUME_ID:/var/lib/cassandra cassandra:latest
```

####  Step 3: Start Docker on the other nodes 

The only difference from the previous docker run command is the addition of the -e CASSANDRA_SEEDS=10.0.0.1 parameter. This is a pointer to the IP address of the first Cassandra node.

On Cassandra node 2 run the following:

```
# docker run --name cassandra2 -d -p 7000:7000 -p 9042:9042 -p 9160:9160 -e CASSANDRA_BROADCAST_ADDRESS=10.0.0.2 -e CASSANDRA_SEEDS=10.0.0.1 -v DOCKER_CREATE_VOLUME_ID:/var/lib/cassandra cassandra:latest
```

On Cassandra node 3 run the following:

```
# docker run --name cassandra3 -d -p 7000:7000 -p 9042:9042 -p 9160:9160 -e CASSANDRA_BROADCAST_ADDRESS=10.0.0.3  -e CASSANDRA_SEEDS=10.0.0.1 -v DOCKER_CREATE_VOLUME_ID:/var/lib/cassandra cassandra:latest
```

Remember to change the IP addresses in our examples to the ones used by your instances. It can take up to 30 seconds for Cassandra to start up on each node. To determine when your cluster is ready for use, view the logs: You should see messages that each node is part of the cluster.

### Walkthrough of Docker Registry with PX-Lite
The Docker Registry is a server-side application that stores and lets you distribute Docker images. The following instructions use Docker
Registry version 2.3.0.

#### Step 1: Create a storage volume for the Docker registry
To create a storage volume for the Docker registry, run the following command on each server and make a note of the returned volume ID. You will need the volume ID when you start the Cassandra container in the next step.

```
# docker volume create -d pxd --opt name=registry_volume --opt size=20000 --opt block_size=64 --opt repl=3 --opt fs=ext4
```

Now we have a volume to attach to our Docker Registry container. The Docker Registry stores its data in the /tmp/registry directory. We will use the Docker -v option to attach the Portworx volume to this directory.

If you don't have the Registry image available locally, you can pull it with docker pull registry.

#### Step 2: Start the Docker Registry
To start the Docker Registry, run the following command. Substitute DOCKER_CREATE_VOLUME_ID for the volume id from the docker volume
create command.

```
# docker run -d -p 5000:5000 --name registry -v DOCKER_CREATE_VOLUME_ID:/tmp/registry registry:2.3.0
```

Your Docker Registry is now available for Docker push and pull commands on port 5000.

## Requirements and Limitations
It is highly recommended that you run PX-Lite on a system with at least 4GB RAM.

|Requirement | Notes |
|---------------|---------|
|Kernel Version|3.10 or higher|
|Docker Version|1.10 or higher|
|KV Database|Etcd.  See https://github.com/coreos/etcd or try a hosted version at https://compose.io/etcd/|
|CPU|4 cores recommended|
|Memory|4GB Minimum|

Other limitations:

| Resource | Limit |
|------------|-------|
| Cluster Size | 3 |
| Per Volume Limit | 100GB |
| Max Volumes | 256 |
| Max local devices | 3 |

