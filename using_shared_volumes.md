PX Shared Volumes allows a common volume/filesystem to be shared by multiple containers across multiple hosts for both read and write activity

The Shared Volumes feature is only available in px-dev version 0.5.4 and above.  To determine your version of px-dev, run 'pxctl -v'.

### Creating shared volumes
To create a Portworx shared volume, use the pxctl command for now.  Shortly, shared volumes can be created through "docker volume create"

```
# pxctl volume create my_shared_vol --shared --size=5 --repl=3
# pxctl volume list
ID			            NAME		        SIZE	   HA	  SHARED	STATUS
944424689751331159	my_shared_vol	  5.0 GiB	 3	  yes	    up - detached
```

Note the "SHARED" status of the volume, in the above output

### Using shared volumes
Shared volumes are used in the same way any external volume would be used in docker.  For example:

```
host1# docker run -it --name box1  -v my_shared_vol:/data --volume-driver=pxd  busybox sh
```

### Using shared volumes from multiple hosts
The same shared volume can be used by different containers on different hosts.   For example:

```
host2# docker run -it --name box1  -v my_shared_vol:/data --volume-driver=pxd  busybox sh
```

Any changes to files and directories will be propagated immediately.   Changes to a shared volume from one host will be immediately visible on all hosts to which the volume is attached.

