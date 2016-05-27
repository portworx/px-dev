Redis can be used as a cache, database, or for messaging. Here is an example of durable storage for containers with container-granular controls. 

In this example, we create a volume for Redis, run and write to Redis, and then snapshot just that Redis volume's state. Portworx snapshots are thinly provisioned and copy-on-write. 

### Create a storage volume for Redis
To create a Portworx storage volume for Redis, use the Docker volume command.

```
# docker volume create -d pxd --name=redis_vol --opt \
     			size=4 --opt block_size=64 --opt repl=1 --opt fs=ext4
```

Now we have a volume to attach to our redis-server container. The redis-server container stores its data in the /data directory. We will use the 'docker -v' option to attach the Portworx volume to this directory.  

### Start the Redis container
To start the Redis container, run the following command, using the 'redis_vol' volume created above. 

```
# docker run --name some-redis  -v redis_vol:/data --volume-driver=pxd  -d redis redis-server --appendonly yes
```
Your redis-server container is now available for for use at port 6379.

### Start the redis CLI client container
To start the redis CLI client container, run the following command.
```
docker run -it --link some-redis:redis --rm redis redis-cli -h redis -p 6379
```

### Populate the redis instance with some data
```
redis:6379> set foo 100
OK
redis:6379> incr foo
(integer) 101
redis:6379> append foo abcxxx
(integer) 9
redis:6379> get foo
"101abcxxx"
redis:6379>
```

### Kill the 'some-redis' instance
```
docker kill some-redis
```

### Start a new redis-server instance with volume persistance
Start another redis-server container called "other-redis using the original 'redis_vol' to show data persistance
```
docker run --name other-redis  -v redis_vol:/data --volume-driver=pxd  -d redis redis-server --appendonly yes
```

### Start a redis CLI container
Connect to the new "other-redis" container with the following command
```
docker run -it --link other-redis:redis --rm redis redis-cli -h redis -p 6379
```

### See that the original data has persisted
```
redis:6379> get foo
"101abcxxx"
```

### Create snapshot of your volume
We can create container-granular snapshots, saving just this container's state. Snapshots are then immedidately available as a volume.  

Create a snapshot of the redis_vol volume using `pxctl`.

```
# pxctl volume list
ID			NAME		SIZE	HA	STATUS
416765532972737036	redis_vol	2.0 GiB	1	up - attached on 3abe5484-756c-4076-8a80-7b7cd5306b28
# pxctl snap create 416765532972737036
Volume successfully snapped:  3291428813175937263
```

### Update the volume with new data
```
# docker run -it --link other-redis:redis --rm redis redis-cli -h redis -p 6379
redis:6379> get foo
"101abcxxx"
redis:6379> set foo foobar
OK
redis:6379> get foo
"foobar"
```

### See that the snapshot volume still contains the original data
```
# docker run --name snap-redis -v 3291428813175937263:/data --volume-driver=pxd -d redis redis-server --appendonly yes
940d2ad6b87df9776e26d29e746eb05fb6081c0e6019d46ba77915d7c8305308
# docker run -it --link snap-redis:redis --rm redis redis-cli -h redis -p 6379
redis:6379> get foo
"101abcxxx"
redis:6379>
```
