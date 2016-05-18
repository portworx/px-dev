These instructions demonstrate running the official `mysql` container from https://hub.docker.com/_/mysql/.
### Watch the video
Here is a 3 minute video that shows setting up a 3 node cluster for mysql and adding more capacity on the fly: https://vimeo.com/163637386

### Create a storage volume for mysql
To create a storage volume for mysql, run the following command and note the returned volume ID. You will need the volume ID when you start the mysql container in the next step.

```
#    docker volume create -d pxd --opt name=mysql_volume --opt \ 
     			size=4 --opt block_size=64 --opt repl=1 --opt fs =ext4
```

Now we have a volume to attach to our mysql container. The mysql container stores its data in the /var/lib/mysql directory. We will use the Docker -v option to attach the Portworx volume to this directory.  

### Start the mysql container
To start the mysql container, run the following command. Substitute DOCKER_CREATE_VOLUME_ID for the volume id from the docker volume create command.

```
# docker run -p 3306:3306 --volume-driver=pxd               \
                        --name pxmysql                      \
                        -e MYSQL_ROOT_PASSWORD=password     \
                        -v DOCKER_CREATE_VOLUME_ID:/var/lib/mysql -d mysql
```
Your mysql container is now available for for use at port 3306.

### Using the `pxctl` CLI to create snaps of your mysql volume
To demonsrate the capabilities of the SAN like functionality offered by px-lite, try creating a snapshot of a mysql volume.

First create a database and a demo table in your mysql container.

```
# mysql --user=root --password=password
MySQL [(none)]> create database pxdemo;
Query OK, 1 row affected (0.00 sec)
MySQL [(none)]> use pxdemo;
Database changed
MySQL [pxdemo]> create table grapevine (counter int unsigned);
Query OK, 0 rows affected (0.04 sec)
MySQL [pxdemo]> quit;
Bye
```

Now create a snapshot of this database using `pxctl`.

```
# /opt/pwx/bin/pxctl snap create 8927336170128555
Volume successfully snapped:  1483421664452964115
```

Note the snapshot volume ID.  Use this to launch a new instance of mysql.  Since you already have mysql running, you can go to another node in your cluster, or stop the original mysql instance.

```
# docker run -p 3306:3306 --volume-driver=pxd 				\
                        --name pxmysqlclone                 \
                        -e MYSQL_ROOT_PASSWORD=password     \
                        -v 1483421664452964115:/var/lib/mysql -d mysql
```

Inspect that the database shows the cloned tables in the new mysql instance. 

```
# mysql --user=root --password=password
MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| pxdemo             |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
MySQL [(none)]> use pxdemo;

Database changed
MySQL [pxdemo]> show tables;
+------------------+
| Tables_in_pxdemo |
+------------------+
| grapevine        |
+------------------+
1 rows in set (0.00 sec)
```
