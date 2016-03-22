These instructions demonstrate running the official `mysql` container from https://hub.docker.com/_/mysql/.

### Step 1: Create a storage volume for mysql
To create a storage volume for mysql, run the following command on each server and make a note of the returned volume ID. You will need the volume ID when you start the mysql container in the next step.

```
    docker volume create -d pxd --opt name=registry_volume --opt \ 
    size=4 --opt block_size=64 --opt repl=1 --opt fs =ext4
```

Now we have a volume to attach to our mysql container. The mysql container stores its data in the /var/lib/mysql directory. We will use the Docker -v option to attach the Portworx volume to this directory.  

### Step 2: Start the mysql container
To start the mysql container, run the following command. Substitute DOCKER_CREATE_VOLUME_ID for the volume id from the docker volume create command.

```
docker run -p 3306:3306 --volume-driver=pxd                 \
                        --name pxmysql                      \
                        -e MYSQL_ROOT_PASSWORD=password     \
                        -v DOCKER_CREATE_VOLUME_ID:/var/lib/mysql -d mysql
```
Your mysql container is now available for for use at port 3306.

Back to [PX-Lite](https://github.com/portworx/px-lite/).
