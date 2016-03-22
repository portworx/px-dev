# Running px-lite with Docker-compose

You can start `px-lite` with `docker-compose` as follows:

```
docker-compose up -d 
```

This will look for the `config.json` file at `/etc/pwx`.

If you do not want to pre-create a `config.json` configuration file, you just start px-lite as follows:

```
docker-compose up -d -daemon --clusterid=YOUR_CLUSTER_ID --devices=/dev/sdb
```
