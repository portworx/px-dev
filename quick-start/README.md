# Running px-lite with Docker Compose

You can start `px-lite` with `docker-compose` as follows:

```
# docker login
pxliteuser

# git clone https://github.com/portworx/px-lite.git
# cd quick-start
# docker-compose run portworx -daemon --kvdb=http://etcd.portworx.com:4001 --clusterid=YOUR_CLUSTER_ID --devices=/dev/xvdi
```

If you have a custom [px configuration file] (https://github.com/portworx/px-lite/edit/master/quick-start/config.json) at `/etc/pwx/config.json`, you can simply start px-lite as follows:

```
# docker-compose up -d 
```
