### Step 1: Launch Servers
First, we create three servers in AWS, using: 
* Image: [Red Han 7.2 (HVM)](https://aws.amazon.com/marketplace/pp/B019NS7T5I/ref=srh_res_product_title?ie=UTF8&sr=0-2&qid=1457648418090)
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
* Following the [Docker install guide for RedHat](https://docs.docker.com/engine/installation/linux/rhel/)
 * Install Docker and start the Docker service
 * Verify that your Docker version is 1.10 or later
  - run ```docker -v ``` in your SSH window
* Configure Docker to use shared mounts 
 - edit the service file for systemd 
  - ```sudo vi /lib/systemd/system/docker.service ``` in your SSH window
  - removed the MountFlags line
 - reload the daemon ```sudo systemctl daemon-reload```
 - restart Docker ```sudo systemctl restart docker```

The shared mounts is required, as PX-Lite exports mount points. 
