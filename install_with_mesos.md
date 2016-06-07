
# Portworx and Mesosphere
Portworx PX-Dev is elastic block storage for containers. Deploying PX-Dev on a server with Docker turns that server into a scale-out storage node. This quick guide shows how to use PX-Dev to implement storage for Mesosphere and Marathon. 

For installing PX-Dev, see our [quick start guides](https://github.com/portworx/px-dev#install-and-quick-start-guides). 

The example below details launching Docker containers through Marathon.   The Marathon application configuration files map options through the Docker command line to reference the Portworx volume driver and associated volume.

## Using PX-Dev with Mesosphere and Marathon
PX-Dev pools your server's capacity and is deployed as a container.   Here is how to use PX-Dev with Mesosphere and Marathon.

### Step 1: Install Mesosphere DC/OS CLI

Follow the instructions for installing [Mesosphere DC/OS](https://dcos.io/install) and the [DC/OS CLI](https://docs.mesosphere.com/1.7/usage/cli/install)

Use the DC/OS CLI command 'dcos node' to identify which nodes in the Mesos cluster are the Agent nodes.

### Step 1: Run the PX-Dev container on each Mesos Agent node

Depending on which base OS is used for the Mesos Agent nodes, launch the PX-Dev container, as per the [examples](https://github.com/portworx/px-dev)

Go through the installation process of creating a PX cluster using each of the Mesos Agent nodes.

### Step 2: Reference PX volumes through Marathon config file

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
              "hostname",
              "UNIQUE"
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

### Step 3: Launch application through Marathon 

Launch the application as you normally would through the DC/OS CLI:

```
dcos marathon app add mysql.json
```
