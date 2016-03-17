
# Docker Compose Quick Start Installation for PX-Lite
If you are using Docker Compose, you can get PX-Lite running by:
 1. Get Docker 1.10 or later. Set to [allow shared mounts](https://github.com/docker/docker/issues/19625).  
   for example: `# mount --make-shared /`
 2. Docker pull portworx/px-lite. [Request access](https://docs.google.com/a/portworx.com/forms/d/1iAhxyIvcDQW7tdrX6UkpjXd5KN9LnWA8UE25n70C-RQ/) during limited preview. 
   Get the [kernel module](https://github.com/portworx/px-lite/blob/master/README.md#kernel-module-for-distros-temporary-requirement)for your distro. 
 3. Get and edit the config
   * `# git clone https://github.com/portworx/px-lite` and open px-lite/config.json 
   *. specify [(cluster ID, etcd URL and storage devices)](https://github.com/portworx/px-lite/blob/master/install_run_ubuntu.md#step-4-edit-the-json-configuration) and save in /etc/pwx/. 
7. `# docker-compose up`

You now have a scale-out storage cluster for containers.
