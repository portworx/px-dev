![logo](http://i.imgur.com/l8JRhxg.jpg)

# PX-Lite alpha
PX-Lite is elastic block storage for containers. Deploying PX-Lite on a server with Docker turns that server into a scale-out, storage node. Storage runs converged on the same server as compute, giving bare metal performance. 

As you develop and deploy your apps in containers, use PX-Lite for elastic storage capacity, managed performance, and high availability.

## Installation and Tutorials
This guide walks through setting up PX-Lite. For the sake of illustration, our example uses Ubuntu on AWS, AWS Elastic Block Storage for storage devices, and a hosted etcd service from Compose.IO. As long as your configuration meets the [Deployment Requirements](https://github.com/portworx/px-lite/#requirements-and-limitations), you can use physical servers, another favorite public cloud, or virtual machines. 

Once this installation is complete, you can continue with walk-throughs for
* [Cassandra storage volumes on PX-Lite](https://github.com/portworx/px-lite/blob/master/cassandra-guide.md)
* [Registry high-availability on PX-Lite](https://github.com/portworx/px-lite/blob/master/registry-guide.md)

## Prerequisites 
PX-Lite requires a server with storage devices, Docker 1.10, and use of a key-value store for the cluster configuration. 

### Step 1: Launch Servers
First, we create three servers in AWS, using: 
* Image: [Ubuntu Server 14.04 LTS (HVM)](https://aws.amazon.com/marketplace/pp/B00JV9JBDS)
* Instance Type: c3.xlarge
* Number of instances: 3
* Storage: 
  - /dev/xvda: 8GB boot device
  - /dev/xvdb: 2GB for container storage
  - /dev/xvdc: 42.9GB for container storage
* (optional) Tag: add value 'px-cluster1' as the name

Volumes used for container data can be magnetic or SSD. PX-Lite will apply different policies based on storage capabilities.

### Step 2: Install and Configure Docker 
* SSH into your first server
* Following the [Docker install guide](https://docs.docker.com/engine/installation/linux/ubuntulinux/)
 * Update your apt sources for Ubuntu Trusty 14.04 (LTS)
  - make sure you use the Trusty 14.04 as the deb entry in step 7
 * Install Docker and start the Docker service
* Verify that your Docker version is 1.10 or later
 - run ```docker -v ``` in your SSH window
* Configure Docker to use shared mounts
 - run ```sudo mount --make-shared / ``` in your SSH window

The shared mounts is required, as PX-Lite exports mount points. If you are using systemd, you would remove the ```MountFlags=slave``` line in your ```docker.service``` file. The Ubuntu image in this example is not using systemd. 

### Step 3: Provision etcd
You can use an existing etcd service or stand-up your own. In this example, we chose [Compose.IO](https://www.compose.io/etcd/) for its ease of use. 

* Create a new etcd deployment in Compose.IO
* Select 256MB RAM as the memory
* Save the connection string, including your username & password
 - example: https://[username]:[password]@[string].dblayer.com:[port]/v2/keys

## Install Portworx PX-Lite 
IMPORTANT: login to the Docker Hub to access PX-Lite, during the limited release period. Contact eric@portworx.com for account access.

### Step 1: Download the PX-Lite Container
From the SSH window for the server:
* Login to Docker Hub 
 * ```# sudo docker login -u [user] -p [password]```
* Pull PX-Lite
 * ```# sudo docker pull portworx/px-lite```

### Step 2: Download and install the PX Kernel Module
This initial version of PX-Lite has a dependency on the [*lightweight*](http://github.com/portworx/px-fuse) kernel module, which must be installed on each server. You can get pre-built packages for select [Centos and Ubuntu Linux](https://github.com/portworx/px-lite#kernel-module-for-varios-distros-temporary-requirement) distributions. 

From the SSH window for the server:
* Download the kernel module for Ubuntu
 * ```# wget http://get.portworx.com/builds/Linux/ubuntu/14.04/px_3.13.0-74_amd64.deb``` 
* Install the kernel module
 * ```# sudo dpkg --install px_3.13.0-74_amd64.deb```

### Step 3: View Disks on Servers (Optional)
PX-Lite pools the storage devices on your local server and creates a global capacity for containers. We will use the two non-root storage devices (```/dev/xvdb```, ```/dev/xvdc```) from our first step in Prerequisites. 

Important: save off any data on storage devices that will be pooled by PX-Lite. Storage devices will be reformatted!

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
 * ```# wget https://github.com/portworx/px-lite/blob/master/conf/config.json```
* Create a directory for the configuration file.
 * ```# sudo mkdir -p /etc/pwx```
* Move the file to that directory. This directory later gets passed in on the Docker command line.
 * ```# sudo cp -p config.json /etc/pwx```
* Edit the config.json to include
 * clusterid: this string identified your cluster and  should be unique within your etcd k/v space
 * kvdb: this is the etcd connection string from the third step in Prerequisites
 * devices: these are the storage devices that will be pooled from the prior step

Example config.json:

```
    {
    "base": {
        "clusterid": "make this unique in your k/v store",
        "kvdb": "https://[username]:[password]@[string].dblayer.com:[port]",
        "storage": {
          "devices": [
            "/dev/xvdb",
            "/dev/xvdc"
         ]
        }
      }
    }
```  
Before running the container, make sure you have saved off any data on the storage devices specified in the config.

      Warning!!!: Any storage device that PX-Lite uses will be reformatted.

## Run PX-Lite 
Through Docker run with PX-Lite, your storage capacity will be aggregated and managed by PX-Lite. As you run PX-Lite on each server, new capacity will be added to the cluster.

Once PX-Lite is up, storage volumes can be created and deleted through the Docker volume commands or pxctl command line tool. With pxctl, you can also inspect volumes, the volume relationships with containers, and nodes. 

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

### Step 2: See Global Capacity 

At this point, PX-Lite should be running on your system. You can run ```Docker ps``` to verify.  

The pxctl control tools are exported to `/opt/pwx/bin/pxctl`. These tools will let you control storage. 

* View the global storage capacity by running
 * ```# sudo /opt/pwx/bin/pxctl status```
 * See the example output below
* Use pxctl to manage volumes, such as create, snapshot, and inspect
 * all pxctl options can be seen by running ```# /opt/pwx/bin/pxctl help```

Output of pxctl status shows the global capacity for Docker containers is now 41 GB. 
```
    # /opt/pwx/bin/pxctl status
    Node ID:  [guid]
    IP:  [ip-address] 
    Local Storage Pool:
    Device		Caching Tier	Size	Used
    /dev/xvdb	true		2.0 GB	2.0 GB
    /dev/xvdc	true		39 GB	1.0 GB
    total		-		41 GB	3.0 GB
```
To increase capacity and enable high-availability, run the same steps on each of the remaining two servers. 


## Using Storage 

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
# /opt/pwx/bin/pxctl create volume foobar
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

# Contact Us
As you use PX-Lite, please share your feedback and ask questions. Find the team on [Google Groups](https://groups.google.com/forum/#!forum/portworx).

# Reference

## Kernel Module for Varios Distros (Temporary Requirement)
If your kernel version is not listed in the table below, you can build the kernel module by following the instructions here: http://github.com/portworx/px-fuse

Find out your kernel version. For example:

```
# uname -r
3.19.3-1.el7.elrepo.x86_64
```
To install pre-built kernel modules, download  and install per on your distro. 

| **Distribution**   | **Kernel** | **Download URL and installation command**                                                                                                                                            |
|  ----------------- | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Centos 7.0         | 3.10.0-229       | [*http://get.portworx.com/builds/Linux/centos/7-3.10.0-229/px-3.10.0-229.14.1.el7.x86\_64.rpm*](http://get.portworx.com/builds/Linux/centos/7-3.10.0-229/px-3.10.0-229.14.1.el7.x86_64.rpm) rpm -ivh px-3.10.0-229.14.1.el7.x86\_64.rpm |
| Centos 7.0         | 3.10.0-327       | [*http://get.portworx.com/builds/Linux/centos/7-3.10.0-327/px-3.10.0-327.10.1.el7.x86\_64.rpm*](http://get.portworx.com/builds/Linux/centos/7-3.10.0-327/px-3.10.0-327.10.1.el7.x86_64.rpm) rpm -ivh px-3.10.0-327.10.1.el7.x86\_64.rpm |
| Centos 7.0         | 3.19.3     | [*http://get.portworx.com/builds/Linux/centos/7-3.19.3/px-3.19.3-1.el7.elrepo.x86\_64.rpm*](http://get.portworx.com/builds/Linux/centos/7-3.19.3/px-3.19.3-1.el7.elrepo.x86_64.rpm) rpm -ivh px-3.19.3-1.el7.elrepo.x86\_64.rpm |
| Ubuntu 14.04       | 3.13       | [*http://get.portworx.com/builds/Linux/ubuntu/14.04/px\_3.13.0-74\_amd64.deb*](http://get.portworx.com/builds/Linux/ubuntu/14.04/px_3.13.0-74_amd64.deb)                     dpkg --install px\_3.13.0-74\_amd64.deb |
| Ubuntu 14.04       | 3.19 (GCE) | [*http://get.portworx.com/builds/Linux/ubuntu/14.04/px\_3.13.0-74\_amd64.deb*](http://get.portworx.com/builds/Linux/ubuntu/14.04-3.19.0-51/px_3.19.0-51_amd64.deb) dpkg --install px_3.19.0-51_amd64.deb |
| Ubuntu 15.04       | 3.19       | [*http://get.portworx.com/builds/Linux/ubuntu/15.04/px\_3.19.0-43\_amd64.deb*](http://get.portworx.com/builds/Linux/ubuntu/15.04/px_3.19.0-43_amd64.deb) dpkg --install px\_3.19.0-43\_amd64.deb |
                                  

## Description of Config.json 

| Field     | Description                                                                                                    | Example                              | Required |
|-----------|----------------------------------------------------------------------------------------------------------------|--------------------------------------|----------|
| ClusterID | A unique identifier for your cluster. Be sure to use the same identifier on all the nodes you want to cluster. | 5ac2ed6f-7e4e-4e1d-8e8c-3a6df1fb61a5 | required |
| mgtiface  | The network interface for management data.                                                                     | eth0                                 | optional |
| dataiface | The network interface for data transfers.                                                                      | eth1                                 | optional |
| kvdb      | The URI to your etcd server.                                                                                   | https://myetcd.example.com:4001      | required |
| devices   | The list of devices that PX-Lite will use. Any disks listed will be reformatted for PX use.                    | /dev/xvda                            | required |

## PX Command Line (pxctl) Help

```
# /opt/pwx/bin/pxctl help
NAME:
   px - px cli

USAGE:
   px [global options] command [command options] [arguments...]
   
VERSION:
   7f1d25e1092c07226a344ab393f18edfbf2d6841
   
COMMANDS:
   show, s      Show volumes and nodes
   create, c    Create volumes
   delete, d    Delete volumes
   inspect, i   Inspect volumes and nodes
   host         Access volumes directly from the host
   service      Service mode utilities
   status       Show status summary
   version      Show version
   help, h      Shows a list of commands or help for one command
   
GLOBAL OPTIONS:
   --json, -j           output in json
   --color              output with color coding
   --raw, -r            raw CLI output for instrumentation
   --help, -h           show help
   --version, -v        print the version
```

## Requirements and Limitations
It is highly recommended that you run PX-Lite on a system with at least 4GB RAM.

|Requirement | Notes |
|---------------|---------|
|Kernel Version|3.10 or higher|
|Docker Version|1.10 or higher|
|KV Database|Etcd 2.0 or higher.  See https://github.com/coreos/etcd or try a hosted version at https://compose.io/etcd/|
|CPU|4 cores recommended|
|Memory|4GB Minimum|
|Cloud|If running in the cloud, AWS Ubuntu 14.04 LTS (HVM) CentOS7 with Updates HVM|
|systemd|If using systemd, Docker should NOT be set to MountFlags=slave.  PX-Lite exports mount points and requires shared mount flags.  Tracking [Docker issue 19625](https://github.com/docker/docker/issues/19625).|

Other limitations:

| Resource | Limit |
|------------|-------|
| Cluster Size | 3 |
| Per Volume Limit | 100GB |
| Max Volumes | 256 |
| Max local devices | 3 |

For more information, visit our [GitHub](https://github.com/portworx/px-lite)

