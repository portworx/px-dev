
# Portworx and Rancher
Portworx PX-Dev is elastic storage fabric for containers. Deploying PX-Dev on a server with Docker turns that server into a scale-out storage node. This quick guide shows how to use PX-Dev to implement storage for Rancher.  The following has been qualified using Rancher v1.1.2, Cattle v0.165.8.

For installing PX-Dev, see our [quick start guides](https://github.com/portworx/px-dev#install-and-quick-start-guides). 

The example below details how to make use of Portworx within Rancher.   

## Using PX-Dev with Rancher
PX-Dev pools your server's capacity and is deployed as a container.   Here is how to use PX-Dev within Rancher.

### Step 1: Install Rancher

Follow the instructions for installing [Rancher](http://docs.rancher.com/rancher/latest/en/quick-start-guide/)

### Step 2: Label Hosts that Run Portworx

If new hosts are added through the GUI, be sure to create a label with the following key-value pair:   "fabric : px"

As directed, copy from the clipboard and paste on to the new host.   The command will be of the following form:
```
sudo docker run -e CATTLE_AGENT_IP="192.168.33.12"  -e CATTLE_HOST_LABELS='fabric=px'  -d --privileged \
           -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/rancher:/var/lib/rancher           \
            rancher/agent:v1.0.2 http://192.168.33.10:8080/v1/scripts/98DD3D1ADD1F0CE368B5:1470250800000:IVpsBQEDjYGHDEULOfGjt9qgA
```
Using IP addresses appropriate for your environment.

Note the CATTLE_HOST_LABELS indicating this node participates in a PX fabric.

### Step 3: Launch Jobs Specifying Host Affinity

When launching new jobs, be sure to include a label, indicating the job's affinity for running on a host with "fabric=px".
The label clause should look like:
```
labels:
    io.rancher.scheduler.affinity:host_label: fabric=px
```

Following is an example for starting up Elasticsearch.  The "docker-compose.yml" file would be:
```
elasticsearch:
  image: elasticsearch:latest
  command: elasticsearch -Des.network.host=0.0.0.0
  ports:
    - "9200:9200"
    - "9300:9300"
  volume_driver: pxd
  volumes:
    - elasticsearch1:/usr/share/elasticsearch/data
  labels:
      io.rancher.scheduler.affinity:host_label: fabric=px
```

Note the 'pxd' volume driver as well as the volume itself (elasticsearch1).

The referenced volume can be a volume name, a volume ID, or a snapshot ID.  

Note:  At this time, Rancher supports docker-compose v1 syntax, but not v2 syntax
