![logo](http://i.imgur.com/l8JRhxg.jpg)

# PX-Lite alpha
PX-Lite is elastic block storage for containers. Deploying PX-Lite on a server with Docker turns that server into a scale-out storage node. Storage runs converged on the same server as compute, giving bare-metal performance. 

PX-Lite aims to improve the storage experience for DevOps teams using containers. This release is an alpha and we want to develop this solution with the community. [Contact us](https://github.com/portworx/px-lite#contact-us) to share your feedback, work with us, and to request features. Stay tuned for updates on PX-Lite and our PX-Enterprise release. 

## Install and Quick Start Guides
As you develop and deploy your apps in containers, use PX-Lite for elastic storage capacity, managed performance, and high availability.

 * See our quick start guides on installing PX-Lite:
  * [Launching PX-Lite in four steps with Docker Compose](https://github.com/portworx/px-lite/blob/master/quick-start/README.md)  
  * [Detailed instructions for installing and Running PX-Lite on Ubuntu](https://github.com/portworx/px-lite/blob/master/install_run_ubuntu.md) or [Red Hat](https://github.com/portworx/px-lite/blob/master/install_run_rhel.md)
 * Run stateful containers with Docker volumes:
  * [Scaling a Cassandra database with PX-Lite](https://github.com/portworx/px-lite/blob/master/examples/cassandra_guide.md) 
  * [Running the Docker registry with high availability](https://github.com/portworx/px-lite/blob/master/examples/registry_guide.md) 
 * Use our [pxctl CLI ](https://github.com/portworx/px-lite/blob/master/cli_reference.md) to directly: 
  * View the cluster global capacity and health
  * Create, inspect, and delete storage volumes
  * Attach policies for IOPs prioritization, maximum volume size, and enable storage replication
 * Refer to the [Technical FAQ and Troubleshooting guide](https://github.com/portworx/px-lite/blob/master/faq.md) if you run into an issue. Please also feel free to [Contact us](https://github.com/portworx/px-lite#contact-us) as well. 
  

## Architecture and Storage
Portworx storage is deployed as a container and runs on a cluster of servers. Application containers provision storage directly through the Docker [volume plugins](https://docs.docker.com/engine/extend/plugins_volume/#command-line-changes:be52bcf493d28afffae069f235814e9f) API or the Docker [command-line](https://docs.docker.com/engine/extend/plugins_volume/#command-line-changes:be52bcf493d28afffae069f235814e9f). Administrators and DevOps can alternatively pre-provision storage through the Portworx command-line tool (pxctl) and then set storage policies using the Portworx administrative interface.

Portworx storage runs in a cluster of server nodes. 
 * Each server has the PX-Lite container and the Docker daemon.
 * Servers join a cluster and share configuration through the key/value store, such as etcd.
 * The PX-Lite container pools the capacity of the storage media residing on the server. You easily select storage media through the config.json file.

See [Deployment Requirements](https://github.com/portworx/px-lite#requirements-and-limitations) for compatibility requirements.

![fig1: storage devices](https://github.com/portworx/px-lite/blob/master/images/cluster.png)

Storage volumes are thinly provisioned, using capacity only as an application consumes it. Volumes are replicated across the nodes within the cluster, per a volume’s configuration, to ensure high availability. 

Using MySQL as an example, a PX-Lite storage cluster has the following characteristics:
 * MySQL is unchanged and continues to write its data to /var/lib/mysql.
 * This data gets stored in the container’s volume, managed by PX-Lite. 
 * PX-Lite synchronously and automatically replicates writes to the volume across the cluster.

![fig2: MySQL volumes](https://github.com/portworx/px-lite/blob/master/images/mysql.png)

Each volume specifies its request of resources (such as its max capacity and IOPS) and its individual requirements (such as ext4 as the file system and block size). 

Using IOPS as an example, a team can chose to set the MySQL container to have a higher IOPS than an offline batch processing container. Thus, a container scheduler can move containers, without losing storage and while protecting the user experience.

# Contact Us
As you use PX-Lite, please share your feedback and ask questions. Find the team on [Google Groups](https://groups.google.com/forum/#!forum/portworx).

# Reference

## Kernel module for distros (Temporary Requirement)
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
| Ubuntu 14.04       | 3.13.0-79 | [*http://get.portworx.com/builds/Linux/ubuntu/14.04/px\_3.13.0-79\_amd64.deb*](http://get.portworx.com/builds/Linux/ubuntu/14.04-3.13.0-79/px_3.13.0-79_amd64.deb) dpkg --install px_3.13.0-79_amd64.deb |
| Ubuntu 14.04       | 3.19 (GCE) | [*http://get.portworx.com/builds/Linux/ubuntu/14.04/px\_3.19.0-51\_amd64.deb*](http://get.portworx.com/builds/Linux/ubuntu/14.04-3.19.0-51/px_3.19.0-51_amd64.deb) dpkg --install px_3.19.0-51_amd64.deb |
| Ubuntu 15.04       | 3.19       | [*http://get.portworx.com/builds/Linux/ubuntu/15.04/px\_3.19.0-43\_amd64.deb*](http://get.portworx.com/builds/Linux/ubuntu/15.04/px_3.19.0-43_amd64.deb) dpkg --install px\_3.19.0-43\_amd64.deb |
                                  

## Description of Config.json 

| Field     | Description                                                                                                    | Example                              | Required |
|-----------|----------------------------------------------------------------------------------------------------------------|--------------------------------------|----------|
| ClusterID | A unique identifier for your cluster. Be sure to use the same identifier on all the nodes you want to cluster. | 5ac2ed6f-7e4e-4e1d-8e8c-3a6df1fb61a5 | required |
| mgtiface  | The network interface for management data.                                                                     | eth0                                 | optional |
| dataiface | The network interface for data transfers.                                                                      | eth1                                 | optional |
| kvdb      | The URI to your etcd server.                                                                                   | https://myetcd.example.com:4001      | required |
| devices   | The list of devices that PX-Lite will use. Any disks listed will be reformatted for PX use.                    | /dev/xvda                            | required |


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
| Per Volume Limit | 1TB |
| Max Volumes | 256 |
| Max local devices | 3 |


