# Using Portworx Storage 
Portworx is elastic block storage for containers. It manages a server's storage devices, joins servers together to form a storage cluster, and integrates up  the application stack with schedulers and containers. Accordingly, the Portworx toolchain operates at each level of hardware and ties the experience together with other toolchains. 

Applications can provision and consume storage through the Docker API. Administrators can pre-provision storage and manage through the Portworx or Docker CLI. 

This guide links to, the already very good, Docker documentation for how to manage Docker volumes. The rest of this guide is dedicated to the Portworx pxctl command line tool. The Portworx tools refer to servers managed by Portworx storage as 'nodes'. A separate document will describe the Portworx RESTful API and management portal. 

# PX command line tool: pxctl
The pxctl command line tool lets you directly provision and manage storage. All operations from pxctl are reflected back into the  containers that use Portworx storage. In addition what is exposed in Docker volumes, the pxctl tool: 
* gives access to Portworx storage-specific features (like cloning a running container's storage)
* shows the connection between containers and their storage volumes, and
* let's you control the Portworx storage cluster (such as adding nodes to the cluster).

The scope of the pxctl command is global to the cluster. Running pxctl from any node within the cluster will therefore show the same global details. The tool also identifies details specific to that node. All Portworx commands can be shown through running [```pxctl help```](https://github.com/portworx/px-lite/blob/master/px_commandline.md#px-command-line-help). 

This current release of the pxctl tools requires privilege. To run as a privileged user, you can sudo as follows:
```
sudo su
```
The pxctl tool is available in the ```/opt/pwx/bin/``` directory. To let you run pxctl without typing the full directory path each time, you can add pxctl to your PATH as follows:
```
export PATH=/opt/pwx/bin:$PATH
```
Now you can just type ```pxctl``` and we're ready to start.

## Status: overall node and cluster status
You can see the total storage capacity through pxctl status. In the example below, a three node cluster has a global capacity of 413 GiB. The node on which we ran the pxctl command contributes 256 GiB to that global capacity.

As nodes join the cluster, pxctl will report the updated global capacity. 

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
## Show: details of resources
Resources are containers, storage volumes, as well as objects that host and connect containers to storage. The show command can be used to see the details of these resources.

```
pxctl show
NAME:
   px show - Show volumes and nodes

USAGE:
   px show command [command options] [arguments...]

COMMANDS:
   cluster	Show cluster details
   disks	Show available disks on this node
   containers	Show volume usage by containers
   volumes, v	Show volumes in the cluster
   snaps	Show volume snapshots in the cluster
   help, h	Shows a list of commands or help for one command
   
OPTIONS:
   --help, -h	show help
```
Running ```show cluster``` returns the current global state of the cluster, including utilization, the number of containers running per node, and the status of the node within the cluster. 

Example of ```show cluster``` for the same three node cluster:
```
# pxctl show cluster
Cluster Information:
Cluster ID: b6f76c07-7725-4451-9704-1867bca3a0b8 Status: STATUS_OK

Nodes in the cluster:
ID                                   MGMT IP       CPU       MEM TOTAL MEM FREE CONTAINERS STATUS
485a9a8e-4811-4399-a8d0-ec65c7dfafbd 10.21.25.218 0.250627  7.8 GB    7.2 GB   5          ok
993a79e2-3597-4f4e-b3f3-f808036e0677 10.21.25.219 N/A       N/A       N/A      N/A        offline
685324a3-21ef-40cf-92cf-60d605f45d65 10.21.25.220 24.937343 7.8 GB    7.3 GB   10         ok
```

In order to view the cluster from a container centric perspective, run ```show containers```. The output lists the running containers by container ID, the container image/name, and the mounted Portworx storage volume. 

Example of ```show containers``` for the same three node cluster:
```
# pxctl show containers
ID           IMAGE        NAMES       VOLUMES            NODE 									STATUS
4b01b7d9ec4b mysql        /clonesql   788684553346073923 485a9a8e-4811-4399-a8d0-ec65c7dfafbd	Up 39 seconds
1557f4d9a605 gourao/px-li /px-lite    N/A                										Up 3 minutes
a2aa17b4edcf google/cadvi /cadvisor   N/A                										Up 8 minutes
e5a00a52e276 mysql        /jeff-mysql 211470040694089666 685324a3-21ef-40cf-92cf-60d605f45d65	Up 15 minutes
d84fc4caf344 portworx/px-li /px-lite    N/A                										Up 2 minutes
81a3f4b95cdf google/cadvi /cadvisor   N/A                										Up 8 minutes
```

# Volume create and options
Storage is durable, elastic, and has fine-grained controls. Volumes are created from the global capacity of a cluster. Capacity and throughput can be expanded by adding a node to the cluster. Storage volumes are protected from hardware and node failures through automatic replication. 

* Durability: replication is set through policy, using the High-Availability setting
 * Each write is synchronously replicated to a quorum set of nodes
 * Any hardware failure means that the replicated volume has the latest acknowledged writes
* Elastic: capacity and throughput can be added at each level, at any time
 * Volumes are thinly provisioned, only using capacity as needed by the container
  * the volume's maximum size can be expanded and contracted, even after data has been written to the volume

A volume can be created before use by its container or by the container directly at runtime. Creating a volume returns the volume's ID. This same volume ID will be returned in Docker commands (such as ```Docker volume ls```) as is shown in pxctl commands. 

Example of creating a volume through pxctl, where the volume ID is returned:
```
# pxctl create volume foobar
3903386035533561360
```
Throughput is controlled per container and can be shared. Volumes have fine-grained control, set through policy.

 * Throughput is set by the Class of Service setting and throughput capacity is pooled
  * Adding a node to the cluster expands the available throughput for reads and writes
  * The best node is selected to service reads, whether that read is from a local storage device or another node's
  * Read throughput is aggregated, where multiple nodes can service one read request in parallel streams
* Fine-grained controls: policies are specified per volume and give full control to storage
 * Policies enforce how the volume is replicated across the cluster, IOPs priority, filesystem, blocksize, and additional parameters described below. 
 * Policies are specified at create time and can be applied to existing volumes. 
 
Policies are set on the volume through the options parameter. They can also be set through a Docker Compose file. Using a Kubernetes Pod spec is upcoming in a future release.

The available options are shown through the --help command, as shown below:
```
# pxctl create volume --help
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
   0.4.2-1c7f1f9
   
COMMANDS:
   show, s      Show volumes and cluster nodes
   create, c    Create volumes
   delete, d    Delete volumes or cluster nodes
   inspect, i   Inspect volumes and cluster nodes
   host         Access volumes directly from the host
   service      Service mode utilities
   status       Show status summary
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
For more on Docker volumes, refer to  [https://docs.docker.com/engine/reference/commandline/volume_create/](https://docs.docker.com/engine/reference/commandline/volume_create/)
