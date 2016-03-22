# Running px-lite with Docker Compose

You can start `px-lite` with `docker-compose` as follows:

```
# docker login
pxliteuser

# docker-compose up -d 
```

This will look for your [px configuration] (https://github.com/portworx/px-lite/edit/master/quick-start/config.json) file `/etc/pwx/config.json`.

If you do not want to pre-create a `config.json` configuration file, you just start px-lite as follows:

```
# docker login
pxliteuser

# docker-compose run portworx -daemon --kvdb=http://etcd.portworx.com:4001 --clusterid=YOUR_CLUSTER_ID --devices=/dev/xvdi
```
