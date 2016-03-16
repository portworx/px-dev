## Installing and Running PX-Lite on Red Hat
This guide takes you through an install from prerequisites through the PX-Lite setup steps. For the sake of illustration, our example uses AWS EC2 for servers in the cluster, AWS Elastic Block Storage for storage devices, and Compose.IO for a hosted etcd service. As long as your configuration meets the [Deployment Requirements](https://github.com/portworx/px-lite/#requirements-and-limitations), you can use physical servers, another favorite public cloud, or virtual machines. 

Once this installation is complete, you can continue with walk-throughs for:
* [Cassandra storage volumes on PX-Lite](https://github.com/portworx/px-lite/blob/master/cassandra_guide.md)
* [Registry high-availability on PX-Lite](https://github.com/portworx/px-lite/blob/master/registry_guide.md)

## Prerequisites 
PX-Lite requires a server with storage devices, Docker 1.10, and use of a key-value store for the cluster configuration. This guide uses RHEL 7.2 as the OS. For Ubuntu, see [this guide](https://github.com/portworx/px-lite/blob/master/install_run_ubuntu.md) for Docker setup with Ubuntu.


### Step 1: Launch servers
First, we create three servers in AWS, using: 
* Image: [Red Hat 7.2 (HVM)](https://aws.amazon.com/marketplace/pp/B019NS7T5I/ref=srh_res_product_title?ie=UTF8&sr=0-2&qid=1457648418090)
* Instance Type: c3.xlarge
* Number of instances: 3
* Storage: 
  - /dev/xvda: 8 GB boot device
  - /dev/xvdb: 2 GB for container storage
  - /dev/xvdc: 42.9 GB for container storage
* (optional) Tag: add value 'px-cluster1' as the name

Volumes used for container data can be magnetic or SSD. PX-Lite will apply different policies based on storage devices capabilities.

### Step 2: Install and configure Docker 
* SSH into your first server.
* Follow the [Docker install guide for RedHat](https://docs.docker.com/engine/installation/linux/rhel/):
 * Install Docker and start the Docker service.
  - Verify that your Docker version is 1.10 or later.
    + Run ```docker -v ```.
* Configure Docker to use shared mounts.
 - Edit the service file for systemd.
   + ```sudo vi /lib/systemd/system/docker.service ``` 
    + Remove the MountFlags line.
 - Reload the daemon ```sudo systemctl daemon-reload```.
 - Restart Docker ```sudo systemctl restart docker```.

The shared mounts configuration is required, as PX-Lite exports mount points. 

### Step 3: Provision etcd
You can use an existing etcd service or stand-up your own. In this example, we chose [Compose.IO](https://www.compose.io/etcd/) for its ease of use. 

* Create a new etcd deployment in Compose.IO.
* Select 256MB RAM as the memory.
* Save the connection string, including your username and password.
 - Example: https://[username]:[password]@[string].dblayer.com:[port]
  - If you are using Compose.IO, the connection string might end with [port]/v2/keys. Please omit the /v2/keys for now. 

You only need to do this etcd step once. You can use the same etcd service for multiple PX-Lite clusters.  

## Install Portworx PX-Lite 
IMPORTANT: Log in to the Docker Hub to access PX-Lite, during the limited release period. Contact eric@portworx.com for account access.

### Step 1: Download the PX-Lite container
From the SSH window for the server:
* Log in to Docker Hub.
 * ```# sudo docker login -u [user] -p [password]```
* Pull PX-Lite.
 * ```# sudo docker pull portworx/px-lite```

### Step 2: Download and install the PX kernel module
This initial version of PX-Lite has a dependency on the [*lightweight*](http://github.com/portworx/px-fuse) kernel module, which must be installed on each server. You can get pre-built packages for select [Centos and Ubuntu Linux](https://github.com/portworx/px-lite#kernel-module-for-varios-distros-temporary-requirement) distributions. 

From the SSH window for the server:
* Download the kernel module for Ubuntu
 * ```curl -O http://get.portworx.com/builds/Linux/centos/7-3.10.0-327/px-3.10.0-327.10.1.el7.x86_64.rpm``` 
* Install the kernel module.
 * ```sudo rpm -ivh px-3.10.0-327.10.1.el7.x86_64.rpm```

### Step 3: View disks on servers (optional)
PX-Lite pools the storage devices on your local server and creates a global capacity for containers. We will use the two non-root storage devices (```/dev/xvdb```, ```/dev/xvdc```) from our first step in Prerequisites. 

Important: Back up any data on storage devices that will be pooled by PX-Lite. Storage devices will be reformatted!

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

### Step 4: Edit the JSON configuration
The PX-Lite `config.json` lets you select the storage devices and identifies the key-value store for the cluster. 

* Download the sample config.json file:
 * ```https://github.com/portworx/px-lite/blob/master/conf/config.json```
* Create a directory for the configuration file.
 * ```# sudo mkdir -p /etc/pwx```
* Move the file to that directory. This directory later gets passed in on the Docker command line.
 * ```# sudo cp -p config.json /etc/pwx```
* Edit the config.json to include the following:
 * clusterid: This string identified your cluster and  should be unique within your etcd k/v space.
 * kvdb: This is the etcd connection string from the third step in Prerequisites.
 * devices: These are the storage devices that will be pooled from the prior step.

Example config.json:

```
    {
      "clusterid": "make this unique in your k/v store",
      "kvdb": "https://[username]:[password]@[string].dblayer.com:[port]",
      "storage": {
        "devices": [
          "/dev/xvdb",
          "/dev/xvdc"
        ]
      }
    }
```  
Before running the container, make sure you have saved off any data on the storage devices specified in the config.

      Warning!!!: Any storage device that PX-Lite uses will be reformatted.

### Step 5: Add nodes

To add  nodes to increase capacity and enable high availability, complete the following steps for each server.

* Repeat Steps 1 and 2 in the Prerequisites section.
 * Launch each server with your Operating System and install Docker.
* Repeat Steps 1 and 2 in the Install Portworx PX-Lite section. 
 * Download the PX-Lite container and install the PX Kernel module on each node.
* JSON configuration:
 * If you have the same device configuration on every node, then copy the  config.json you created the first time in Step 4 to all the nodes. 
 * If you have different device configurations on the nodes, then repeat Steps 3 and 4 in the Install Portworx PX-Lite section. Use the same clusterid and kvdb on all the nodes. 

Afterwards, continue with [Using PX-Lite storage](https://github.com/portworx/px-lite/blob/master/px_commandline.md).

## Run PX-Lite 
When you run Docker and PX-Lite, your storage capacity is aggregated and managed by PX-Lite. As you run PX-Lite on each server, new capacity is added to the cluster.

Once PX-Lite is running, you can create and delete storage volumes through the Docker volume commands or the pxctl command line tool. With pxctl, you can also inspect volumes, the volume relationships with containers, and nodes. 

### Step 1: Run PX-Lite
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

Explanation of the runtime command options:

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

### Step 2: View global capacity 

At this point, PX-Lite should be running on your system. You can run ```Docker ps``` to verify.  

The pxctl control tools are exported to `/opt/pwx/bin/pxctl`. These tools  let you control storage. For more on using pxctl, see [Using Portworx storage](https://github.com/portworx/px-lite/blob/master/px_commandline.md).

* View the global storage capacity by running
 * ```# sudo /opt/pwx/bin/pxctl status```
 * See the example output below
* Use pxctl to manage volumes, for example to create, snapshot, and inspect.
 * View all pxctl options by running ```# /opt/pwx/bin/pxctl help```

The following output of pxctl status shows that the global capacity for Docker containers is now 41 GB. 
```
    # /opt/pwx/bin/pxctl status
    Status: PX is operational
    Node ID:  c553a764-9565-4f6b-b70c-10d963096b76
     	IP:  172.31.23.134 
     	Local Storage Pool:
     	Device		Caching Tier	Size	Used
     	/dev/xvdf	true            64 GB   2.0 GB
     	/dev/xvdg	true            64 GB   2.0 GB
     	total		-               128 GB  4.0 GB
    Cluster Summary
     	ID:  86428a61-a22a-4a7a-a79a-1cfbbb846f7e
     	IP: 172.31.23.134 - Capacity: 119 GiB/3.7 GiB OK (This node)
    Global Storage Pool
     	Total Capacity	:  119 GiB
     	Total Used    	:  3.7 GiB
```

You have now completed setup of PX-Lite on your first server. To increase capacity and enable high availability, repeat the same steps on each of the remaining two servers. Run pxctl status to view the cluster status. Afterwards, continue  with [Quick Start Guides](https://github.com/portworx/px-lite/blob/master/README.md#install--and-quick-start-guides) for application scenarios that use PX-Lite.
