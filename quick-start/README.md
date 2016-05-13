# Running PX-Dev with Docker Compose

You can start `PX-Dev` with `docker-compose` as follows:

```

# git clone https://github.com/portworx/px-dev.git
# cd quick-start
# docker-compose run portworx -daemon --kvdb=http://myetcd.example.com:4001 --clusterid=YOUR_CLUSTER_ID --devices=/dev/xvdi
```

If you have a custom [px configuration file] (https://github.com/portworx/px-dev/edit/master/quick-start/config.json) at `/etc/pwx/config.json`, you can simply start px-dev as follows:

```
# docker-compose up -d 
```

You now have a scale-out storage cluster for containers. Continue with more examples and [Quick Start Guides](https://github.com/portworx/px-dev/blob/master/README.md#install-and-quick-start-guides). 

This release is an alpha and we want to develop this solution with the community. [Contact us](https://github.com/portworx/px-dev#contact-us) to share your feedback, work with us, and to request features. Stay tuned for updates on PX-Lite and our PX-Enterprise release. 
