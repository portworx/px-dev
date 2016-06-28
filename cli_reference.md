# Using Portworx Storage 
Portworx is elastic storage for containers. `px-dev` provides a distributed hyper-converged storage software for Docker.  It manages a server's storage devices, joins servers together to form a storage cluster, and integrates up the application stack with schedulers and containers. Accordingly, the Portworx toolchain operates at each level of hardware and ties the experience together with other toolchains.

Applications can provision and consume storage through the Docker API. Administrators can pre-provision storage and manage through the Portworx or Docker CLI. 

This guide links to the already very good Docker documentation for how to manage Docker volumes. The rest of this guide is dedicated to the Portworx pxctl command line tool. The Portworx tools refer to servers managed by Portworx storage as nodes. A separate document will describe the Portworx RESTful API and management portal. 

# PX Control Tool: pxctl
The pxctl tool lets you directly provision and manage storage. All operations from pxctl are reflected back into the  containers that use Portworx storage. In addition to what is exposed in Docker volumes, the pxctl tool: 
* Gives access to Portworx storage-specific features (like cloning a running container's storage)
* Shows the connection between containers and their storage volumes
* Let you control the Portworx storage cluster (such as adding nodes to the cluster)

The scope of the pxctl command is global to the cluster. Running pxctl from any node within the cluster will therefore show the same global details. The tool also identifies details specific to that node. All Portworx commands can be shown through running [```pxctl help```](https://github.com/portworx/px-dev/blob/master/cli_reference.md#px-command-line-help). 

This current release of the pxctl tools requires privilege. To run as a privileged user, you can sudo as follows:
```
sudo su
```
The pxctl tool is available in the ```/opt/pwx/bin/``` directory. To let you run pxctl without typing the full directory path each time, you can add pxctl to your PATH as follows:
```
export PATH=/opt/pwx/bin:$PATH
```
Now you can just type ```pxctl``` and you're ready to start.

## Status: Overall Node and Cluster Status
You can see the total storage capacity through pxctl status. In the example below, a three-node cluster has a global capacity of 413 GiB. The node on which we ran the pxctl command contributes 256 GiB to that global capacity.

As nodes join the cluster, pxctl reports the updated global capacity. 

Example of the status summary from the first node:
```
# pxctl status
Status: PX is operational
Node ID:  2ecf6b47-c461-4f80-b334-55954eb229fb
        IP:  10.21.25.218 
        Local Storage Pool:
        Device          Caching Tier    Size    Used
        /dev/xvdj       true            128 GB  4.0 GB
        /dev/xvdi       true            128 GB  4.0 GB
        total           -               256 GB  4.0 GB
Cluster Summary
        ID:  px_cluster_1
        IP: 10.21.25.218 - Capacity: 256 GiB/1.9 GiB OK (This node)
        IP: 10.21.25.219 - Capacity: 186 GiB/1.9 GiB OK
        IP: 10.21.25.220 - Capacity: 186 GiB/1.9 GiB OFFLINE
Global Storage Pool
        Total Capacity  :  413 GiB
        Total Used      :  3.7 GiB
```
## Volumes: Manage storage volumes
Use this command to create and manage volumes.  These volumes can directly be used with Docker with the `-v` option.

```
NAME:
   pxctl volume - Manage volumes

USAGE:
   pxctl volume command [command options] [arguments...]

COMMANDS:
   create, c	Create a volume
   list, l	List volumes in the cluster
   inspect, i	Inspect a volume
   delete, d	Delete a volume
   stats, st	Volume Statistics
   alerts, a	Show volume related alerts
   help, h	Shows a list of commands or help for one command
   
OPTIONS:
   --help, -h	show help
```
Running ```cluster list``` returns the current global state of the cluster, including utilization, the number of containers running per node, and the status of the node within the cluster. 

Example of ```cluster list``` for the same three-node cluster:
```
# pxctl cluster list
Cluster ID: cluster-xxx-yyy-zzz
Status: OK

Nodes in the cluster:
ID                                      MGMT IP         CPU             MEM TOTAL       MEM FREE        CONTAINERS      STATUS
8018cc5a-8293-49ef-904c-600b3f562ef2    172.31.25.219   1.754386        7.8 GB          6.6 GB          N/A             ok
75b37f58-7ef1-4b2d-acc4-37d3ceb5b30a    172.31.25.218   0.375           7.8 GB          7.2 GB          N/A             ok
7707a0cb-eda0-4f9a-921a-c778e2d722df    172.31.25.220   0               7.8 GB          6.4 GB          N/A             ok
```

To view the cluster from a container-centric perspective, run ```container show```. The output lists the running containers by container ID, the container image/name, and the mounted Portworx storage volume. 

Example of ```container show``` for the same three-node cluster:
```
# pxctl container show
ID           IMAGE        NAMES       VOLUMES            NODE 									STATUS
4b01b7d9ec4b mysql        /clonesql   788684553346073923 485a9a8e-4811-4399-a8d0-ec65c7dfafbd	Up 39 seconds
1557f4d9a605 gourao/px-li /px-dev    N/A                										Up 3 minutes
a2aa17b4edcf google/cadvi /cadvisor   N/A                										Up 8 minutes
e5a00a52e276 mysql        /jeff-mysql 211470040694089666 685324a3-21ef-40cf-92cf-60d605f45d65	Up 15 minutes
d84fc4caf344 portworx/px-li /px-dev    N/A                										Up 2 minutes
81a3f4b95cdf google/cadvi /cadvisor   N/A                										Up 8 minutes
```

# Volume Create and Options
Storage is durable, elastic, and has fine-grained controls. Portworx creates volumes from the global capacity of a cluster. You can expand capacity and throughput by adding a node to the cluster. Portworx protects storage volumes from hardware and node failures through automatic replication. 

* Durability: Set replication through policy, using the High Availability setting.
 * Each write is synchronously replicated to a quorum set of nodes.
 * Any hardware failure means that the replicated volume has the latest acknowledged writes.
* Elastic: Add capacity and throughput at each layer, at any time.
 * Volumes are thinly provisioned, only using capacity as needed by the container.
  * You can expand and contract the volume's maximum size, even after data has been written to the volume.

A volume can be created before use by its container or by the container directly at runtime. Creating a volume returns the volume's ID. This same volume ID is returned in Docker commands (such as ```Docker volume ls```) as is shown in pxctl commands. 

Example of creating a volume through pxctl, where the volume ID is returned:
 ```
 # pxctl volume create foobar
  3903386035533561360
 ```
Throughput is controlled per container and can be shared. Volumes have fine-grained control, set through policy.

 * Throughput is set by the Class of Service setting and throughput capacity is pooled.
  * Adding a node to the cluster expands the available throughput for reads and writes.
  * The best node is selected to service reads, whether that read is from a local storage devices or another node's storage devices.
  * Read throughput is aggregated, where multiple nodes can service one read request in parallel streams.
* Fine-grained controls: policies are specified per volume and give full control to storage.
 * Policies enforce how the volume is replicated across the cluster, IOPs priority, filesystem, blocksize, and additional parameters described below. 
 * Policies are specified at create time and can be applied to existing volumes. 
 
Set policies on a volume through the options parameter. Or, set policies through a Docker Compose file. Using a Kubernetes Pod spec is slated for a future release.

Show the available options through the --help command, as shown below:
```
# pxctl volume create --help
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

## PX Command Line Help

```
# pxctl help
NAME:
   pxctl - px cli

USAGE:
   pxctl [global options] command [command options] [arguments...]
   
VERSION:
   0.4.3-9da1bcd
   
COMMANDS:
   status       Show status summary
   volume, v    Manage volumes
   snap, s      Manage volume snapshots
   cluster, c   Manage the cluster
   container    Display containers in the cluster
   service, sv  Service mode utilities
   host         Attach volumes to the host
   eula         Show license agreement
   help, h      Shows a list of commands or help for one command
   
GLOBAL OPTIONS:
   --json, -j           output in json
   --color              output with color coding
   --raw, -r            raw CLI output for instrumentation
   --help, -h           show help
   --version, -v        print the version
```
## Volumes with Docker
All Docker volume commands are reflected into Portworx storage. For example, a ```Docker volume create``` command will provision a storage volume in a Portworx storage cluster.  

```
# docker volume create -d pxd --name <volume_name>
```
As part of the volume command, you can add optional parameters through the --opt flag. The option parameters are the same, whether you use Portworx storage through the Docker volume or the pxctl commands. 

Example of options for selecting the container's filesystem and volume size: 

```
  docker volume create -d pxd --name <volume_name> --opt fs=ext4 --opt size=10G
```
For more on Docker volumes, refer to  [https://docs.docker.com/engine/reference/commandline/volume_create/](https://docs.docker.com/engine/reference/commandline/volume_create/).


