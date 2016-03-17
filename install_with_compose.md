
## Docker Compose Quick Start Installation for PX-Lite
If you are using Docker Compose, you can get PX-Lite running by:

 1. Get Docker 1.10 or later. 
   * Set to [allow shared mounts](https://github.com/docker/docker/issues/19625). For example: `# mount --make-shared /`.
 2. Docker pull portworx/px-lite. [Request access](https://docs.google.com/a/portworx.com/forms/d/1iAhxyIvcDQW7tdrX6UkpjXd5KN9LnWA8UE25n70C-RQ/) during the limited preview.
   * Get the [kernel module](https://github.com/portworx/px-lite/blob/master/README.md#kernel-module-for-distros-temporary-requirement) for your distro. 
 3. Get and specify the config for your cluster.
   * `# git clone https://github.com/portworx/px-lite` 
   * `cd px-lite` 
   * edit config.json and save into /etc/pwx/
     * Specify the cluster ID, an etcd service, and the [storage devices](https://github.com/portworx/px-lite/blob/master/install_run_ubuntu.md#step-4-edit-the-json-configuration).
 4. `# docker-compose up` and PX-Lite is now running.

You now have a scale-out storage cluster for containers. Continue with more examples and [Quick Start Guides](https://github.com/portworx/px-lite/blob/master/README.md#install-and-quick-start-guides). 

This release is an alpha and we want to develop this solution with the community. [Contact us](https://github.com/portworx/px-lite#contact-us) to share your feedback, work with us, and to request features. Stay tuned for updates on PX-Lite and our PX-Enterprise release. 

