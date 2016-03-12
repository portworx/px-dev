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

## Show: View Status State
You can see the overall state of your cluster. XXX
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
