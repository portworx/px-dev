
# Portworx and Mesosphere
Portworx PX-Dev is elastic storage fabric for containers. Deploying PX-Dev on a server with Docker turns that server into a scale-out storage node. This quick guide shows how to use PX-Dev to implement [persistent storage for Mesosphere DC/OS and Marathon](https://portworx.com/use-case/persistent-storage-dcos/).  The following has been qualified using DC/OS 1.7.

For installing PX-Dev, see our [quick start guides](https://github.com/portworx/px-dev#install-and-quick-start-guides). 

The example below details launching Docker containers through Marathon.   The Marathon application configuration files map options through the Docker command line to reference the Portworx volume driver and associated volume.

## Using PX-Dev with Mesosphere and Marathon
PX-Dev pools your server's capacity and is deployed as a container.   Here is how to use PX-Dev with Mesosphere and Marathon.

### Step 1: Install Mesosphere DC/OS CLI

Follow the instructions for installing [Mesosphere DC/OS](https://dcos.io/install) and the [DC/OS CLI](https://docs.mesosphere.com/1.7/usage/cli/install)

Use the DC/OS CLI command 'dcos node' to identify which nodes in the Mesos cluster are the Agent nodes.

### Step 2: Run the PX-Dev container on Mesos Agent nodes

Depending on which base OS is used for the Mesos Agent nodes, launch the PX-Dev container, as per the [examples](https://github.com/portworx/px-dev)

Go through the installation process of creating a PX cluster using Mesos Agent nodes.

### Step 3: Add Mesos 'constraints'
For each Mesos Agent node that is participating in the PX cluster, specify MESOS_ATTRIBUTES that allow for affinity of tasks to nodes that are part of the PX cluster

- Add "MESOS_ATTRIBUTES=fabric:px" to the file /var/lib/dcos/mesos-slave-common
- Restart the slave service
  - rm -f /var/lib/mesos/slave/meta/slaves/latest
  - systemctl restart dcos-mesos-slave.service
- Verify the slave service has started properly
  - systemctl status dcos-mesos-slave.service

### Step 4: Reference PX volumes through Marathon config file

The 'pxd' docker volume driver and any associated volumes are passed in to Marathon as docker parameters.   The following example illustrates for 'mysql'.
```
{
    "id": "mysql",
    "cpus": 0.5,
    "mem": 256,
    "instances": 1,
    "container": {
        "type": "DOCKER",
        "docker": {
            "image": "mysql:5.6.27",
            "parameters": [
                    {
                       "key": "volume-driver",
                       "value": "pxd"
                    },
                    {
                       "key": "volume",
                       "value": "mysql_vol:/var/lib/mysql"
                    }],
            "network": "BRIDGE",
              "portMappings": [
                {
                  "containerPort": 3306,
                  "hostPort": 32000,
                  "protocol": "tcp"
                }
                ]
        }
    },
    "constraints": [
            [
              "fabric",
              "CLUSTER",
              "px"
            ]],
    "env": {
        "MYSQL_ROOT_PASSWORD": "password"
    },
      "minimumHealthCapacity" :0,
      "maximumOverCapacity" : 0.0
}
```


Note the docker 'parameters' clause as the way of referencing the 'pxd' volume driver as well as the volume itself.

The referenced volume can be a volume name, a volume ID, or a snapshot ID.   If the volume name does not previously exist, then it will be created in-band with default settings.

Note the 'constraints' clause, which restricts this task to running only on agent nodes that are part of the PX cluster.

### Step 3: Launch application through Marathon 

Launch the application as you normally would through the DC/OS CLI:

```
dcos marathon app add mysql.json
```
