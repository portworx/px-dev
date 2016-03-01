![logo](http://i.imgur.com/l8JRhxg.jpg)

# PX-Lite alpha

PX-Lite aggregates the storage capacity of hard drives on your server and it clusters multiple servers for high availability. As you develop and deploy your apps in containers, use PX-Lite for elastic storage capacity, managed performance, and high availability.  For more information, visit our [GitHub](https://github.com/portworx/px-lite)

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
| Centos 7.0         | 3.10.0-229       | [*http://get.portworx.com/builds/Linux/centos/7-3.10.0-229/px-3.10.0-229.14.1.el7.x86\_64.rpm*](http://get.portworx.com/builds/Linux/centos/7-3.10.0-229/px-3.10.0-229.14.1.el7.x86_64.rpm) rpm -ivh px-3.10.0-229.14.1.el7.x86\_64.rpm |
| Centos 7.0         | 3.10.0-327       | [*http://get.portworx.com/builds/Linux/centos/7-3.10.0-327/px-3.10.0-327.10.1.el7.x86\_64.rpm*](http://get.portworx.com/builds/Linux/centos/7-3.10.0-327/px-3.10.0-327.10.1.el7.x86_64.rpm) rpm -ivh px-3.10.0-327.10.1.el7.x86\_64.rpm |
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
# wget https://raw.githubusercontent.com/portworx/px-lite/master/conf/config.json
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

| Field     | Description                                                                                                    | Example                              | Required |
|-----------|----------------------------------------------------------------------------------------------------------------|--------------------------------------|----------|
| ClusterID | A unique identifier for your cluster. Be sure to use the same identifier on all the nodes you want to cluster. | 5ac2ed6f-7e4e-4e1d-8e8c-3a6df1fb61a5 | required |
| mgtiface  | The network interface for management data.                                                                     | eth0                                 | optional |
| dataiface | The network interface for data transfers.                                                                      | eth1                                 | optional |
| kvdb      | The URI to your etcd server.                                                                                   | https://myetcd.example.com:4001      | required |
| devices   | The list of devices that PX-Lite will use. Any disks listed will be reformatted for PX use.                    | /dev/xvda                            | required |


To find local drives that are available for use on your system, you can issue this bash command:
```
# fdisk -l
```

*Warning: Please ensure that disks are empty, to avoid data loss!*                                                                                                                                                                 
 
### Step 4: Running PX-Lite
IMPORTANT: Remember to login to the Docker hub to access this private repository.  Please contact eric@portworx.com for account access.

```
# docker login -u user -p password
```

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

*WARNING: If using systemd, Docker should NOT be set to MountFlags=slave. PX-Lite exports mount points and requires shared mount flags. [Tracking Docker issue 19625](https://github.com/docker/docker/issues/19625).*     

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

### Step 5: Access the PX CLI

At this point, PX should be running on your system.  You can access the CLI at `/opt/pwx/bin/pxctl`:
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

### Trouble?
See if your issue has been posted at our [Google Groups](https://groups.google.com/forum/#!forum/portworx).

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

