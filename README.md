# PX-Lite alpha

PX-Lite aggregates the storage capacity of hard drives on your server and it clusters multiple servers for high availability. As you develop and deploy your apps in containers, use PX-Lite for elastic storage capacity, managed performance, and high availability.

### Quick-start guides on running PX-Lite for:
    -   Hosting WordPress with persistent storage
    -   Scaling a Cassandra database
    -   Running the Docker registry with high availability

### CLI quick-start guide for tools to directly manage volumes, such as container granular snapshots and clones:
    -   CLI quick-start
    -   Example: Snapshotting a container’s storage. We will add docs to our
    -   CLI RESTful interface

PX-Lite improves the experience for DevOps teams. We want to develop this solution with the community. [*Contact*](#h.2wz3tlxiekfq)[*u*](#h.2wz3tlxiekfq)[*s*](#h.2wz3tlxiekfq) to share your feedback, work with us, and to request features. Stay tuned for updates on PX-Lite and our PX-Enterprise release.


## Getting started

PX-Lite aggregates the storage capacity on a server and it clusters servers for high availability. Containerized applications provision storage directly through the Docker volume plugin API command-line or admins can pre-provision storage through the Portworx CLI, called `pxctl`.  When the `px-lite` container is run, the CLI is automatically available at the host at `/opt/pwx/bin/pxctl`.

PX-Lite runs as a Docker container, available on the DockerHub. This initial version of PX-Lite has a dependency on the [*lightweight*](http://github.com/portworx/px-fuse) kernel module, which must be installed on hosts.

The following guides walk through setting up and using PX-Lite and maintaining storage. See the Deployment Requirements for details on the configuration.

### Prerequisites

#### Docker
Docker version 1.10 or greater is required and can be obtained [here](https://docs.docker.com/engine/installation/binaries/).

### Linux Kernel
A 3.10 Linux kernel is the minimum requirement for px-lite.

### Machine configuration
A minimum machine configuration of 4GB RAM and 4 cores is highly recommended

### A key value database
A key value database such as etcd or consul is required. To setup etcd, follow these instructions:
1. https://github.com/coreos/etcd OR
2. https://quay.io/repository/coreos/etcd

### Portworx Driver
Currently the portworx px storage driver requires a kernel module.

On `centos` for example, this module can be installed the following way:

```
# wget http://wilkins.portworx.com:8080/job/PX-FUSE/lastSuccessfulBuild/artifact/go/src/github.com/portworx/px-fuse/rpm/out/px-3.19.3-1.el7.elrepo.15.x86_64.rpm
# rpm -ivh px-3.19.3-1.el7.elrepo.15.x86_64.rpm
```

You can download and install pre-built packages for select Centos and Ubuntu Linux distributions. If your kernel version is not listed in the table below, you can build the kernel module by following the instructions here: http://github.com/portworx/px-fuse

Find out your kernel version. For example:
```
# uname -r
# 3.19.3-1.el7.elrepo.x86\_64
```

#### Finding the correct module for your kernel.

| Distribution 	| Kernel 	| Download URL                                                                                                                                                        	|
|--------------	|--------	|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| Centos 7.0   	| 3.10   	| `http://get.portworx.com/builds/Linux/centos/7/px-3.10.0-229.14.1.el7.x86_64.rpm`                           	|
| Centos 7.0   	| 3.19.3 	| `http://get.portworx.com/builds/Linux/centos/7-3.19.3/px-3.19.3-1.el7.elrepo.x86_64.rpm`             	|
| Ubuntu 14.04 	| 3.13   	| `http://get.portworx.com/builds/Linux/ubuntu/15.04/px_3.19.0-43_amd64.debdpkg` 	|


##### Step 2: Install Docker

PX-Lite requires
[*Docker*](https://docs.docker.com/engine/installation/) version 1.10 or
later. If you are using systemd, make sure that the Docker daemon is
configured to let PX-Lite export storage.

##### Step 3: Edit the JSON configuration

The PX-Lite config.json specifies the key-value store for the cluster
and lets you select which storage devices PX-Lite will use.

To download the sample config.json file:

  wget http://get.portworx.com/px-lite/config.json
  --------------------------------------------------

Create a directory for the configuration file and move the file to that
directory. This directory later gets passed in on the Docker command
line.

  ---------------------------------
  sudo mkdir -p /etc/pwx

  sudo cp -p config.json /etc/pwx
  ---------------------------------
  ---------------------------------

Here is a sample config.json file.

  -----------------------------------------
  {

  "version": "0.4",

  "base": {

  "clusterid": "CLUSTERID",

  "mgtiface": "eth0",

  "kvdb": "http://etcd.example.com:4001",

  "storage": {

  "devices": \[

  "/dev/DEVICE\_ID\_1",

  "/dev/DEVICE\_ID\_2"

  \],

  }

  }

  }
  -----------------------------------------
  -----------------------------------------

In the configuration file, make the CLUSTERID unique among clusters in
your key-value store. For example: px-lite-cluster-1. Point the
DEVICE\_ID\_N to a local unused block device on your system, for example
/dev/sdb. Any storage device that PX-Lite uses will be reformatted.

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Field       Description                                                                                                      Example                           Required
  ----------- ---------------------------------------------------------------------------------------------------------------- --------------------------------- ----------
  version     Release version of PX-Lite.                                                                                      0.4                               required

  clusterid   A unique identifier for your cluster. Be sure to use the same identifier on all the nodes you want to cluster.   px-lite-cluster-1                 required

  mgtiface    The network interface for management data.                                                                       eth0                              optional

  dataiface   The network interface for data transfers.                                                                        eth1                              optional

  kvdb        The URI to your etcd server.                                                                                     https://myetcd.example.com:4001   required

  storage     Outer data structure for holding devices and future configuration.                                                                                 required

  devices     The list of devices that PX-Lite will use. Any disks listed will be reformatted for PX use.                      /dev/xvda                         required
                                                                                                                                                                 
              Warning: Please ensure that disks are empty, to avoid data loss.                                                 /dev/xvdf                         
                                                                                                                                                                 
                                                                                                                               /dev/xvdg                         
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------

##### Step 4: Download the PX-Lite container

Download the PX-Lite container with the following command:

  docker pull portworx/px-lite:latest
  -------------------------------------

#### Running PX-Lite

Start the PX-Lite container with the following run command:

  -------------------------------------------------------------------------------
  docker run --restart=always --name px-lite -d --net=host --privileged=true \\

  -v /run/docker/plugins:/run/docker/plugins \\

  -v /var/lib/osd:/var/lib/osd:shared \\

  -v /dev:/dev \\

  -v /etc/pwx:/etc/pwx \\

  -v /opt/pwx/bin:/export\_bin:shared \\

  -v /var/run/docker.sock:/var/run/docker.sock \\

  --ipc=host portworx/px-lite:latest
  -------------------------------------------------------------------------------
  -------------------------------------------------------------------------------

run command options:

--privileged

> Sets PX-Lite to be a privileged container. Required to export block
> device and for other functions.

--net=host

> Sets communication to be on the host IP address over ports 9001 -
> 9003. Future versions will support separate IP addressing for PX-Lite.

--shm-size=384M

> PX-Lite advertises support for asynchronous I/O. It uses shared memory
> to sync across process restarts

Volume flags:

> Note: The format on the command line is
> /\[host\_path\]/\[container\_path\]. For readability in this
> description, we omit the second path when the two paths are identical.

-v /run/docker/plugins

> Specifies that the volume driver interface is enabled.

-v /dev

> Specifies which host drives PX-Lite can see. Note that PX-Lite only
> uses drives specified in config.json. This volume flage is an
> alternate to --device=\[\].

-v /etc/pwx/config.json:/etc/pwx/config.json

> the configuration file location.

-v /var/run/docker.sock

> Used by Docker to export volume container mappings.

-v /var/lib/osd:/var/lib/osd:shared

> Location of the exported container mounts. This must be a shared
> mount.

-v /opt/pwx/bin:/export\_bin:shared

> Exports the PX command line (pxctl) tool from the container to the
> host.

#### Create volume

With Docker:

  -----------------------------------------------------------------------------------------------------------------------------------------------
  [*https://docs.docker.com/engine/reference/commandline/volume\_create/*](https://docs.docker.com/engine/reference/commandline/volume_create/)

  docker volume create -d pxd --name &lt;volume\_name&gt;

  Everything else is optional.

  Use --opt to specify optional parameters; use the same option keywords as pxctl.

  EXAMPLE:

  docker volume create -d pxd --name &lt;volume\_name&gt; --opt fs=ext4 --opt size=10G
  -----------------------------------------------------------------------------------------------------------------------------------------------
  -----------------------------------------------------------------------------------------------------------------------------------------------

With pxctl:

  -----------------------------------------------------------------------------
  pxctl create volume -h

  NAME:

  create volume - Create a volume

  USAGE:

  command create volume \[command options\] \[arguments...\]

  OPTIONS:

  --label, -l

  > Comma separated name=value pairs. Example: name=sqlvolume,type=production

  --size, -s "10G"

  > Volume size in GBs.

  --fs "ext4"

  > File system layout: none|xfs|ext4

  --seed

  > Optional. Data to seed the volume.

  --block\_size, -b "32"

  > Volume block size in KBs.

  --repl, -r "1"

  > Replication factor \[1..2\]

  --cos "1"

  > Class of Service: \[1..9\]

  --snap\_interval, --si "0"

  > Snapshot interval in minutes. 0 disables snapshots.
  -----------------------------------------------------------------------------
  -----------------------------------------------------------------------------

### Walkthrough of Cassandra with PX-Lite

Apache Cassandra is an open source distributed database management
system designed to handle large amounts of data across commodity
servers.

Setting up a Cassandra cluster with Portworx storage takes only a few
commands. In this scenario we will create a three-node Cassandra
cluster.

#### Step 1: Create storage volumes for each instance

To create storage volumes for each instance, run the following command
on each server.

  -------------------------------------------------------------------
  docker volume create -d pxd --opt name=cassandra\_volume --opt \\

  size=20000 --opt block\_size=64 --opt repl=1 --opt fs=ext4
  -------------------------------------------------------------------
  -------------------------------------------------------------------

The output of the command is the volume identifier. Store this for
later; you will need the identifier to start the containers.

Before you start the Docker containers, you also need to tell Cassandra
what IP address to advertise to other Cassandra nodes. In this scenario
we will use 10.0.0.1 as the IP address for the first Cassandra node;
10.0.0.2, for the second Cassandra node, and 10.0.0.3, for the third
Cassandra node.

#### Step 2: Start the Cassandra Docker image on node 1

We will use the docker -v option to assign the volume we created with
docker volume create. Substitute the DOCKER\_CREATE\_VOLUME\_ID for the
volume ID that was returned from docker volume create. You should also
substitute your IP address for the 10.0.0.1 placeholder in the
CASSANDRA\_BROADCAST\_ADDRESS parameter.

  ---------------------------------------------------------------------------------------------------------------------
  docker run --name cassandra1 -p 7000:7000 -p 9042:9042 -p \\ 9160:9160 -e CASSANDRA\_BROADCAST\_ADDRESS=10.0.0.1 \\

  -v DOCKER\_CREATE\_VOLUME\_ID:/var/lib/cassandra cassandra:latest
  ---------------------------------------------------------------------------------------------------------------------
  ---------------------------------------------------------------------------------------------------------------------

####  Step 3: Start Docker on the other nodes 

The only difference from the previous docker run command is the addition
of the -e CASSANDRA\_SEEDS=10.0.0.1 parameter. This is a pointer to the
IP address of the first Cassandra node.

On Cassandra node 2 run the following:

  -------------------------------------------------------------------
  docker run --name cassandra2 -d -p 7000:7000 -p 9042:9042 \\

  -p 9160:9160 -e CASSANDRA\_BROADCAST\_ADDRESS=10.0.0.2 \\

  -e CASSANDRA\_SEEDS=10.0.0.1 \\

  -v DOCKER\_CREATE\_VOLUME\_ID:/var/lib/cassandra cassandra:latest
  -------------------------------------------------------------------
  -------------------------------------------------------------------

On Cassandra node 3 run the following:

  -------------------------------------------------------------------
  docker run --name cassandra3 -d -p 7000:7000 -p 9042:9042 \\

  -p 9160:9160 -e CASSANDRA\_BROADCAST\_ADDRESS=10.0.0.3 \\

  -e CASSANDRA\_SEEDS=10.0.0.1 \\

  -v DOCKER\_CREATE\_VOLUME\_ID:/var/lib/cassandra cassandra:latest
  -------------------------------------------------------------------
  -------------------------------------------------------------------

Remember to change the IP addresses in our examples to the ones used by
your instances. It can take up to 30 seconds for Cassandra to start up
on each node. To determine when your cluster is ready for use, view the
logs: You should see messages that each node is part of the cluster.

### Walkthrough of Docker Registry with PX-Lite

The Docker Registry is a server-side application that stores and lets
you distribute Docker images. The following instructions use Docker
Registry version 2.3.0.

#### Step 1: Create a storage volume for the Docker registry

To create a storage volume for the Docker registry, run the following
command on each server and make a note of the returned volume ID. You
will need the volume ID when you start the Cassandra container in the
next step.

  docker volume create -d pxd --opt name=registry\_volume --opt \\ size=20000 --opt block\_size=64 --opt repl=3 --opt fs =ext4
  ------------------------------------------------------------------------------------------------------------------------------

Now we have a volume to attach to our Docker Registry container. The
Docker Registry stores its data in the /tmp/registry directory. We will
use the Docker -v option to attach the Portworx volume to this
directory.

If you don't have the Registry image available locally, you can pull it
with docker pull registry.

#### Step 2: Start the Docker Registry

To start the Docker Registry, run the following command. Substitute
DOCKER\_CREATE\_VOLUME\_ID for the volume id from the docker volume
create command.

  ------------------------------------------------------------
  docker run -d -p 5000:5000 --name registry \\

  -v DOCKER\_CREATE\_VOLUME\_ID:/tmp/registry registry:2.3.0
  ------------------------------------------------------------
  ------------------------------------------------------------

Your Docker Registry is now available for Docker push and pull commands
on port 5000.

### Deployment Requirements

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Requirement       Details or Version                                                                                                                          Comments
  ----------------- ------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------------------------------------
  Docker Version    v1.10                                                                                                                                       requires volume plug-ins

  Kernel            3.10+                                                                                                                                       temporary requirement.

  key value store   etcd version 2.0                                                                                                                            

  Memory            2 GB for PX-Lite                                                                                                                            temporary requirement for 2GB of overhead

  OS                [*AWS Ubuntu 14.04 LTS (HVM)*](https://aws.amazon.com/marketplace/pp/B00JV9TBA6/ref=sp_mpg_product_title?ie=UTF8&sr=0-5)                    AWS is not required
                                                                                                                                                                
                    Ubuntu XXX                                                                                                                                  
                                                                                                                                                                
                    [*CentOS7 with Updates HVM*](https://aws.amazon.com/marketplace/pp/B00O7WM7QW/ref=srh_res_product_title?ie=UTF8&sr=0-2&qid=1453413879042)   

  systemd           If using systemd, Docker should NOT be set to [*MountFlags*](https://github.com/docker/docker/issues/19625)=slave                           PX-Lite exports mount points and requires shared mount flags.
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
