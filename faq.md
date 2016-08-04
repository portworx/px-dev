## Technical FAQ and Troubleshooting
1. Getting logs for PX-Dev
  Access logs like any other Docker container. Example:
    * ```docker ps ``` and get the CONTAINER ID for px-dev
    * ```docker logs [CONTAINER_ID] ```

2. No such file or directory, when running on SELinux 
 If you have `SELinux` enabled, you may get the following error message: 
 ```
 # docker run --name mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw  --volume-driver=pxd -v sql_vol:/var/lib/mysql -d mysql
 docker: Error response from daemon: no such file or directory.
 See 'docker run --help'.
 ```
 This can be remedied as follows:
 ```
 # docker run  --security-opt=label:disable  --name mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw  --volume-driver=pxd -v  sql_vol:/var/lib/mysql -d mysql
 ```
 You will not need to workaround this after [20834](https://github.com/docker/docker/pull/20834) is merged

3. Pxctl show persmission denied
 If you run a pxctl command and get the error like the below, the issue is that you need to run as *root* to use the pxctl tools. 
 ```
 pxctl cluster list
 show cluster: Get http://unix.sock/v1/cluster/enumerate: dial unix /var/lib/osd/cluster/osd.sock: connect: permission denied
  ```
 To enable root, you can run  ```sudo su ```. 

4. Invalid value: mode shared mode shown on PX-Dev run

 The error below is when your Docker version is not 1.10 or greater. Example, running with v1.9.1 will report this error.
  ```
 invalid value "/var/lib/osd:/var/lib/osd:shared" for flag -v: bad mode specified: shared
  ```
5.  Not enough free space in /dev/shm

Example: 
```
"Invalid PX configuration: Configuration check failed: Not enough free space in /dev/shm, needs 258MB, available 224MB"
```
  To fix:
```
mount -o remount,size=1GB /dev/shm
```

