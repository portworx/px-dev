![logo](http://i.imgur.com/l8JRhxg.jpg)

[![Docker Pulls](https://img.shields.io/docker/pulls/portworx/px-dev.svg)](https://hub.docker.com/r/portworx/px-dev)
# PX-Developer

PX-Developer (PX-Dev) is scale-out storage and data services for containers.  PX-Dev itself is deployed as a container with your application stack.  By running PX-Dev with your application stack, you get container-granular controls for storage persistence, capacity management, performance, and availability in a scaleout environment. Deploying the PX-Developer container on a server with Docker Engine turns that server into a scale-out storage node. Storage runs converged with compute and gives bare-metal drive performance. PX-Dev can be used alongside [Docker](https://portworx.com/use-case/docker-persistent-storage/) to provide persistent storage for containerized applications to get familiarized with running persistent storage use-cases for containerized applications. Portworx recommends using PX-Enterprise for production installations for all orchestrators including Kubernetes, DC/OS, Nomad, Docker EE and more.

PX-Dev offers container granular services such as:

1. Data persistence in a multi node environment
2. Synchronous data availability across multiple availability zones and automatic AZ detection
3. Automatic tiering and class of service enforcement
4. Bring-your-own-key Encryption
5. Shared namespaces across containers running on different servers
6. S3 interfaces and backup to S3
7. Integration with schedulers to automate container placement

Please visit [our offical docs site for more information on running Portworx](https://docs.portworx.com)

Visit our website to learn more about some of the most common use cases:

[Docker persistent storage](https://portworx.com/use-case/docker-persistent-storage/)
[Kubernetes storage](https://portworx.com/use-case/kubernetes-storage/)
[DCOS persistent storage](https://portworx.com/use-case/persistent-storage-dcos/)

Join us on slack @ http://slack.portworx.com/


