# Using Portworx Storage 
Portworx is elastic block storage for containers. It spans from a server’s hardware, across servers to form a cluster, and up to the application stack to integrate with schedulers and containers. Accordingly, the toolchain operates at each aspect of the stack and integrates with other toolchains. 

Applications can provision and consume storage through the Docker API. Administrators can pre-provision storage and manage through the Portworx or Docker CLI. 

This guide links to, the already very good, Docker documentation for how manage Docker volumes. The rest of this guide is dedicated to the Portworx pxctl command line tool. A separate document will describe the Portworx RESTful API and management portal. 

## Volumes with Docker
All Docker volume commands are reflected into Portworx storage. For example, a ```Docker volume create``` command will provision a storage volume in Portworx.  

```
# docker volume create -d pxd --name <volume_name>
```
As part of the volume command, you can add optional parameters through the --opt flag. The option parameters are the same, whether you use Portworx storage through the Docker volume or the pxctl commands. 

Example of options for selecting the container's filesystem and volume size: 

```
  docker volume create -d pxd --name <volume_name> --opt fs=ext4 --opt size=10G
```
For more on Docker volumes, refer to  [https://docs.docker.com/engine/reference/commandline/volume_create/](https://docs.docker.com/engine/reference/commandline/volume_create/)

# PX command line tool pxctl
The pxctl command line tool lets you directly provision and manage storage. All operations from pxctl are reflected back into the  containers that use Portworx storage. In addition what is exposed in Docker volumes, the pxctl gives access to Portworx storage-specific features (like cloning a running container's storage) and let's you control  the Portworx storage cluster (such as adding servers to the cluster).

All Portworx commands can be shown through running ```pxctl help``` as shown below.

To be able to access the pxctl from any working directory, you can add pxctl to your PATH as follows:
```
export PATH=/opt/pwx/bin:$PATH
```

## Status: View overall node and cluster status
You can see the total storage capacity through pxctl status. As servers join the cluster, pxctl will report show the increased global capacity. 

Example of capacity from one server:
```
 pxctl status
 Node ID:  [guid]
	IP:  172.31.17.65 
 	Local Storage Pool:
	Device		Caching Tier	Size	  Used
	/dev/xvdb	true  			8.0 GB    2.0 GB
	/dev/xvdc	true			12 GB	  2.0 GB
	total		-		 		20 GB	  4.0 GB
 Cluster Summary
	ID:  cluster 2
	IP: 172.31.17.65	Capacity:    20 GB/  4.0 GB OK (This node)
 Global Storage Pool
	Total Capacity	:  20 GB
	Total Used    	:  4.0 GB
```

## Show: Show volumes and nodes.
You can see the overall state of your volumes and cluster nodes.
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

### Show cluster: Show the nodes in the cluster.
```
# pxctl show cluster
Cluster Information:
Cluster ID: b6f76c07-7725-4451-9704-1867bca3a0b8 Status: STATUS_OK

Nodes in the cluster:
ID                                   MGMT IP       CPU       MEM TOTAL MEM FREE CONTAINERS STATUS
485a9a8e-4811-4399-a8d0-ec65c7dfafbd 102.21.25.218 0.250627  7.8 GB    7.2 GB   5          ok
993a79e2-3597-4f4e-b3f3-f808036e0677 102.21.25.219 N/A       N/A       N/A      N/A        offline
685324a3-21ef-40cf-92cf-60d605f45d65 102.21.25.220 24.937343 7.8 GB    7.3 GB   10         ok
```

### Show containers: Show the containers in the cluster and the volumes they are using.
```
# pxctl show containers
```
# pxctl show containers
ID           IMAGE        NAMES       VOLUMES            NODE 									STATUS
4b01b7d9ec4b mysql        /clonesql   788684553346073923 485a9a8e-4811-4399-a8d0-ec65c7dfafbd	Up 39 seconds
1557f4d9a605 gourao/px-li /px-lite    N/A                										Up 3 minutes
a2aa17b4edcf google/cadvi /cadvisor   N/A                										Up 8 minutes
e5a00a52e276 mysql        /jeff-mysql 211470040694089666 685324a3-21ef-40cf-92cf-60d605f45d65	Up 15 minutes
d84fc4caf344 gourao/px-li /px-lite    N/A                										Up 2 minutes
81a3f4b95cdf google/cadvi /cadvisor   N/A                										Up 8 minutes
```

## Create a volume
```
# pxctl create volume foobar
3903386035533561360
```

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

### Seed option: volume with initial data
A useful pattern is to be able to provide initial data in newly created volumes. In pxctl, you can do so with the seed command. 

XXXX

### Priortization option: Class of Service
As container run, their IOPs will be priorted based on the level set on the volume. The level can be changed on a running container. A higher number has greater priority. 
XXXX

## PX Command Line (pxctl) Help

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
