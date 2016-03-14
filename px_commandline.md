
# Using Storage 

## Creating a volume with Docker:

Refer to  [https://docs.docker.com/engine/reference/commandline/volume_create/](https://docs.docker.com/engine/reference/commandline/volume_create/)

```
# docker volume create -d pxd --name <volume_name>
```
Everything else is optional. Use --opt to specify optional parameters; use the same option keywords as pxctl.

For Example: 

```
  docker volume create -d pxd --name <volume_name> --opt fs=ext4 --opt size=10G
```

## Creating a volume with pxctl:
```
# /opt/pwx/bin/pxctl create volume foobar
3903386035533561360
```

```
# /opt/pwx/bin/pxctl create volume --help
NAME:
   create volume - Create a volume

USAGE:
   command create volume [command options] [arguments...]

OPTIONS:
   --label, -l                  Comma separated name=value pairs, e.g name=sqlvolume,type=production
   --size, -s "1000"            specify size in MB
   --fs "ext4"                  filesystem to be laid out: none|xfs|ext4
   --seed                       optional data that the volume should be seeded with
   --block_size, -b "32"        block size in Kbytes
   --repl, -r "3"               replication factor [1..3]
   --cos "1"                    Class of Service: [1..9]
   --snap_interval, --si "0"    snapshot interval in minutes, 0 disables snaps

```

## PX Command Line (pxctl) Help

```
# /opt/pwx/bin/pxctl help
NAME:
   pxctl - px cli

USAGE:
   pxctl [global options] command [command options] [arguments...]
   
VERSION:
   0.4.2-1c7f1f9
   
COMMANDS:
   show, s      Show volumes and cluster nodes
   create, c    Create volumes
   delete, d    Delete volumes or cluster nodes
   inspect, i   Inspect volumes and cluster nodes
   host         Access volumes directly from the host
   service      Service mode utilities
   status       Show status summary
   eula         Show license agreement
   help, h      Shows a list of commands or help for one command
   
GLOBAL OPTIONS:
   --json, -j           output in json
   --color              output with color coding
   --raw, -r            raw CLI output for instrumentation
   --help, -h           show help
   --version, -v        print the version
```
