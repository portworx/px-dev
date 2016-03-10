### Step 1: Launch Servers
First, we create three servers in AWS, using: 
* Image: [Red Hat 7.2 (HVM)](https://aws.amazon.com/marketplace/pp/B019NS7T5I/ref=srh_res_product_title?ie=UTF8&sr=0-2&qid=1457648418090)
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
* Follow the [Docker install guide for RedHat](https://docs.docker.com/engine/installation/linux/rhel/)
 * Install Docker and start the Docker service
  - Verify that your Docker version is 1.10 or later
    + run ```docker -v ``` 
* Configure Docker to use shared mounts 
 - edit the service file for systemd 
   + ```sudo vi /lib/systemd/system/docker.service ``` 
    + remove the MountFlags line
 - reload the daemon ```sudo systemctl daemon-reload```
 - restart Docker ```sudo systemctl restart docker```

The shared mounts is required, as PX-Lite exports mount points. 

### Step 3: Provision etcd
You can use an existing etcd service or stand-up your own. In this example, we chose [Compose.IO](https://www.compose.io/etcd/) for its ease of use. 

* Create a new etcd deployment in Compose.IO
* Select 256MB RAM as the memory
* Save the connection string, including your username & password
 - example: https://[username]:[password]@[string].dblayer.com:[port]

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
 * ```curl -O http://get.portworx.com/builds/Linux/centos/7-3.10.0-327/px-3.10.0-327.10.1.el7.x86_64.rpm``` 
* Install the kernel module
 * ```sudo rpm -ivh px-3.10.0-327.10.1.el7.x86_64.rpm```

