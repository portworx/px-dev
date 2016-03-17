```
# pxctl 
NAME:
   pxctl - px cli

USAGE:
   pxctl [global options] command [command options] [arguments...]
   
VERSION:
   0.4.3-d07d5d1
   
COMMANDS:
   status       Show status summary
   volume, v    Manage volumes
   snap, s      Manage volume snapshots
   cluster, c   Manage the cluster
   container    Display containers in the cluster
   service, sv  Service mode utilities
   host         Attach volumes to the host
   eula         Show license agreement
   help, h      Shows a list of commands or help for one command
   
GLOBAL OPTIONS:
   --json, -j           output in json
   --color              output with color coding
   --raw, -r            raw CLI output for instrumentation
   --help, -h           show help
   --version, -v        print the version
   

```

```
# pxctl volume
NAME:
   pxctl volume - Manage volumes

USAGE:
   pxctl volume command [command options] [arguments...]

COMMANDS:
   create, c    Create a volume
   show, s      Show volumes in the cluster
   inspect, i   Inspect a volume
   delete, d    Delete a volume
   stats, s     Volume Statistics
   alerts, a    Show volume related alerts
   help, h      Shows a list of commands or help for one command
   
OPTIONS:
   --help, -h   show help
   
```
