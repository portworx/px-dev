
## Installing and Running PX-Dev on Ubuntu
This guide takes you through an install from prerequisites through the PX-Dev setup steps. For the sake of illustration, our example uses AWS EC2 for servers in the cluster, AWS Elastic Block Storage for storage devices, and Compose.IO for a hosted etcd service. As long as your configuration meets the [Deployment Requirements](https://github.com/portworx/px-dev/#requirements-and-limitations), you can use physical servers, another favorite public cloud, or virtual machines. 

Once this installation is complete, you can continue with walk-throughs for:
* [Cassandra storage volumes on PX-Dev](https://github.com/portworx/px-dev/blob/master/examples/cassandra_guide.md)
* [Registry high-availability on PX-Dev](https://github.com/portworx/px-dev/blob/master/examples/registry_guide.md)

## Prerequisites 
PX-Dev requires a server with storage devices, Docker 1.10, and use of a key-value store for the cluster configuration. This guide uses Ubuntu as the OS. For RedHat, see [this guide](https://github.com/portworx/px-dev/blob/master/install_run_rhel.md) for Docker setup with RedHat, including configuring systemd.

### Step 1: Launch servers
First, we create three servers in AWS, using: 
* Image: [Ubuntu Server 14.04 LTS (HVM)](https://aws.amazon.com/marketplace/pp/B00JV9JBDS)
* Instance Type: c3.xlarge
* Number of instances: 3
* Storage: 
  - /dev/xvda: 8 GB boot device
  - /dev/xvdb: 64 GB for container storage
  - /dev/xvdc: 64 GB for container storage
* (optional) Tag: add value 'px-cluster1' as the name

Volumes used for container data can be magnetic or SSD. PX-Dev will apply different policies based on storage devices capabilities.

### Step 2: Install and configure Docker 
* SSH into your first server
* Following the [Docker install guide](https://docs.docker.com/engine/installation/linux/ubuntulinux/)
 * Update your apt sources for Ubuntu Trusty 14.04 (LTS).
  - Make sure you use the Trusty 14.04 as the deb entry in step 7 when you add an entry for Ubuntu.
 * Install Docker and start the Docker service.
* Verify that your Docker version is 1.10 or later.
 - Run ```docker -v ``` in your SSH window
* Configure Docker to use shared mounts
 - Run ```sudo mount --make-shared / ``` in your SSH window

The shared mounts configuration is required, as PX-Dev exports mount points. If you are using systemd, remove the ```MountFlags=slave``` line in your ```docker.service``` file. The Ubuntu image in this example is not using systemd. 

### Step3: Provision etcd
You can use an existing etcd service or stand-up your own. In this example, we chose [Compose.IO](https://www.compose.io/etcd/) for its ease of use. 

* Create a new etcd deployment in Compose.IO.
* Select 256 MB RAM as the memory.
* Save the connection string, including your username and password.
 - Example: https://[username]:[password]@[string].dblayer.com:[port]
  - If you are using Compose.IO, the connection string might end with [port]/v2/keys. Please omit the /v2/keys for now. 

You only need to do this etcd step once. You can use the same etcd service for multiple PX-Dev clusters.  

## Specify Storage
### Step 1: View disks on servers 
PX-Dev pools the storage devices on your local server and creates a global capacity for containers. We will use the two non-root storage devices (```/dev/xvdb```, ```/dev/xvdc```) from our first step in Prerequisites. 

Important: Back up any data on storage devices that will be pooled by PX-Dev. Storage devices will be reformatted!

To view the storage devices on your server: 
* Command line: run ```# lsblk``` 
 * Note the devices without the part(ition)

Example output:

  ```
    $ lsblk
    NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    xvda                      202:0    0     8G  0 disk 
    └─xvda1                   202:1    0     8G  0 part /
    xvdb                      202:16   0     2G  0 disk 
    xvdc                      202:32   0    40G  0 disk
  ```

### Step 2: Edit the JSON configuration
The PX-Dev `config.json` lets you select the storage devices and identifies the key-value store for the cluster. 

* Download the sample config.json file:
 * ```https://github.com/portworx/px-dev/blob/master/conf/config.json```
* Create a directory for the configuration file.
 * ```# sudo mkdir -p /etc/pwx```
* Copy the file to that directory. This directory later gets passed in on the Docker command line.
 * ```# sudo cp -p config.json /etc/pwx```
* Edit the config.json to include the following:
 * clusterid: This string identified your cluster and  should be unique within your etcd k/v space.
 * kvdb: This is the etcd connection string from the third step in Prerequisites.
 * devices: These are the storage devices that will be pooled from the prior step.

Example config.json:

```
    {
      "clusterid": "make this unique in your k/v store",
      "kvdb": [
          "etcd:https://[username]:[password]@[string].dblayer.com:[port]"
        ],
      "storage": {
        "devices": [
          "/dev/xvdb",
          "/dev/xvdc"
        ]
      }
    }
```  

The format for the `kvdb` section is as follows:
```
kvdb:[
  "etcd:[http/https]://[....]"
]
```

 IMPORTANT: If you are using Compose.IO, the kvdb string might end with [port]/v2/keys. Please omit the /v2/keys for now.
 Before running the container, make sure you have saved off any data on the storage devices specified in the config.

      Warning!!!: Any storage device that PX-Dev uses will be reformatted.

## Run PX-Dev 
When you run Docker and PX-Dev, your storage capacity is aggregated and managed by PX-Dev. As you run PX-Dev on each server, new capacity is added to the cluster.

Once PX-Dev is running, you can create and delete storage volumes through the Docker volume commands or the pxctl command line tool. With pxctl, you can also inspect volumes, the volume relationships with containers, and nodes. 

### Step 1: Run PX-Dev
Start the PX-Dev container with the following run command:
```
# sudo docker run --restart=always --name px-dev -d --net=host --privileged=true \
                 -v /run/docker/plugins:/run/docker/plugins                       \
                 -v /var/lib/osd:/var/lib/osd:shared                              \
                 -v /dev:/dev                                                     \
                 -v /etc/pwx:/etc/pwx                                             \
                 -v /opt/pwx/bin:/export_bin:shared                               \
                 -v /var/run/docker.sock:/var/run/docker.sock                     \
                 -v /var/cores:/var/cores                                         \
                 -v /usr/src:/usr/src                                             \
                 --ipc=host                                                       \
                portworx/px-dev
```

Explanation of the runtime command options:

    --privileged
        > Sets PX-Dev to be a privileged container. Required to export block  device and for other functions.
        
    --net=host
        > Sets communication to be on the host IP address over ports 9001 -9003. Future versions will support separate IP addressing for PX-Dev.
        
    --shm-size=384M
        > PX-Dev advertises support for asynchronous I/O. It uses shared memory to sync across process restarts
        
    -v /run/docker/plugins
        > Specifies that the volume driver interface is enabled.
        
    -v /dev
        > Specifies which host drives PX-Dev can see. Note that PX-Dev only uses drives specified in config.json. This volume flage is an alternate to --device=\[\].
        
    -v /etc/pwx/config.json:/etc/pwx/config.json
        > the configuration file location.
        
    -v /var/run/docker.sock
        > Used by Docker to export volume container mappings.
        
    -v /var/lib/osd:/var/lib/osd:shared
        > Location of the exported container mounts. This must be a shared mount.
        
    -v /opt/pwx/bin:/export_bin:shared
        > Exports the PX command line (pxctl) tool from the container to the host.

### Step 2: View global capacity 

At this point, PX-Dev should be running on your system. You can run ```Docker ps``` to verify.  

The pxctl control tools are exported to `/opt/pwx/bin/pxctl`. These tools  let you control storage. For more on using pxctl, see [Using Portworx storage](https://github.com/portworx/px-dev/blob/master/cli_reference.md).

* View the global storage capacity by running
 * ```# sudo /opt/pwx/bin/pxctl status```
 * See the example output below
* Use pxctl to manage volumes, for example to create, snapshot, and inspect.
 * View all pxctl options by running ```# /opt/pwx/bin/pxctl help```

The following output of pxctl status shows that the global capacity for Docker containers is now 128 GB. 
```
    # /opt/pwx/bin/pxctl status
    Status: PX is operational
    Node ID:  c553a764-9565-4f6b-b70c-10d963096b76
     	IP:  172.31.23.134 
     	Local Storage Pool:
     	Device		Caching Tier	Size	Used
     	/dev/xvdb	true            64 GB   2.0 GB
     	/dev/xvdc	true            64 GB   2.0 GB
     	total		-               128 GB  4.0 GB
    Cluster Summary
     	ID:  86428a61-a22a-4a7a-a79a-1cfbbb846f7e
     	IP: 172.31.23.134 - Capacity: 119 GiB/3.7 GiB OK (This node)
    Global Storage Pool
     	Total Capacity	:  119 GiB
     	Total Used    	:  3.7 GiB
```

You have now completed setup of PX-Dev on your first server. To increase capacity and enable high availability, repeat the same steps on each of the remaining two servers. Run pxctl status to view the cluster status. Afterwards, continue  with [Quick Start Guides](https://github.com/portworx/px-dev/blob/master/README.md#install--and-quick-start-guides) for application scenarios that use PX-Dev.


### Step 3: Add nodes
To add nodes to increase capacity and enable high availability, complete the same steps on each server.

* JSON configuration:
 * If you have the same storage device configuration on every node, then copy the  config.json you created the first time to all the nodes. 
 * If you have different storage device configurations on the nodes, make sure to use the same clusterid and kvdb on all the nodes. 

Afterwards, continue with [Using PX-Dev storage](https://github.com/portworx/px-dev/blob/master/cli_reference.md).

