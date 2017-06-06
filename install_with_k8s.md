
# Portworx and Kubernetes
Portworx PX-Dev is elastic storage fabric for containers. Deploying PX-Dev on a server with Docker turns that server into a scale-out storage node. This quick guide shows how to use PX-Dev to implement storage for Kubernetes pods. 

Future versions will support launching PX-Dev through Kubernetes. For more on PX-Dev, see our [quick start guides](https://github.com/portworx/px-dev#install-and-quick-start-guides). 

## Using PX-Dev with Kubernetes
PX-Dev pools your servers capacity and is deployed as a container. Here is how to install PX-Dev on each server. We are tracking when shared mounts will be allowed within Kubernetes (K8s), which will allow Kubernetes to deploy PX-Dev. 

### Step 1: Run the PX-Dev container outside of Kubernetes

Run PX-Dev container using docker with following command

```
$ docker run --restart=always --name px-dev -d --net=host \
--privileged=true \
-v /run/docker/plugins:/run/docker/plugins \
-v /var/lib/osd:/var/lib/osd \
-v /dev:/dev \
-v /etc/pwx:/etc/pwx \
-v /usr/src:/usr/src \
-v /opt/pwx/bin:/export_bin \
-v /usr/libexec/kubernetes/kubelet-plugins/volume/exec/px~flexvolume:/export_flexvolume:shared \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /var/cores:/var/cores \
-v /var/lib/kubelet:/var/lib/kubelet:shared \
--ipc=host \
portworx/px-dev:latest
```

### Step 2: Install Flexvolume Binary on all Kubernetes nodes
Flexvolume allows other volume drivers outside of Kubernetes to
attach/detach/mount/unmount custom volumes to pods/daemonsets/rcs

When you run the Px-Dev container on a kubernetes node, it automatically
installs the flexvolume binary at the required path and is ready to use.

### Step 3: Include PX Flexvolume as a VolumeSpec in Kubernetes spec file

Under "spec" section of your spec yaml file, add a "volumes" section.

``` yaml
spec:
  volumes:
    - name: test
      flexVolume:
        driver: "px/flexvolume"
        fsType: "ext4"
        options:
          volumeID: "615055680017358399"
          size: "1G"
          osdDriver: "pxd"
```
* The driver name should be set to px/flexvolume.
* Specify the unique ID for the volume create in px-dev container as
the volumeID field.
* Set the osdDriver to "pxd" always. It indicates that the flexvolume
should use the px driver for managing volumes.

### Step 4: Include the Flexvolume as a VolumeMount spec in your container/application.

Once you have specified flexvolume as a volume type in your spec
file, you can mount it by including "volumeMounts" spec

Here is an example how you could use it in your container

``` yaml
spec:
  containers:
  - name: nginx
    image: nginx
    securityContext:
      privileged: true
    volumeMounts:
    - name: test
      mountPath: /data
```

* Use the same "name" field as used while defining the volume.

### Step 5: Run the Kubernetes cluster in privileged mode.

* In order to share the namespace between the host, PX-Dev container
  and your Kubernetes pod instance you need to run the cluster with 
  privileges. This can be done by setting the environment variable
  "ALLOW_PRIVILEGED" equal to "true"
* Share the host path "/var/lib/kubelet" with PX-Dev container and
  your pods. The above docker run command for PX-Dev shares this
  path. In order to share it within your pod add a new "hostPath" type
  volume and a corresponding volumeMount in your spec file.

```yaml
spec:
  containers:
    volumeMounts:
    - name: lib
      mountPath: /var/lib/kubelet
  volumes:
    - name: lib
      hostPath:
        path: /var/lib/kubelet

```


## Summary of steps

* Run px-dev container using docker with following command

```
$ docker run --restart=always --name px-dev -d --net=host
--privileged=true \
-v /run/docker/plugins:/run/docker/plugins \
-v /var/lib/osd:/var/lib/osd \
-v /dev:/dev \
-v /etc/pwx:/etc/pwx \
-v /usr/src:/usr/src \
-v /opt/pwx/bin:/export_bin \
-v /usr/libexec/kubernetes/kubelet-plugins/volume/exec/px~flexvolume:/export_flexvolume:shared \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /var/cores:/var/cores \
-v /var/lib/kubelet:/var/lib/kubelet:shared \
--ipc=host \
-p 9001:9001 \
-p 9007:9007 \
-p 9008:9008 \
-p 2345:2345 \
portworx/px-dev:latest
```

* Start your k8s cluster in privileged mode

```
$ cd kubernetes
$ ALLOW_PRIVILEGED=true hack/local-up-cluster.sh
```

* Set your cluster details

```
$ cluster/kubectl.sh config set-cluster local --server=http://127.0.0.1:8080 --insecure-skip-tls-verify=true
$ cluster/kubectl.sh config set-context local --cluster=local
$ cluster/kubectl.sh config use-context local
```

* Run your pod

```
$ ./kubectl create -f nginx-pxd.yaml
```
