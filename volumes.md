# BUILDING A VOLUME:
The volume will be distributed if no stripe or replica options are specified.

Create a distributed volume:
```
gluster volume create firstvol servera:/bricks/brick-a1/brick serverb:/bricks/brick-b1/brick
```
Start newly created volume, to make it accessible to clients
```
gluster volume start firstvol
gluster volume status firstvol
```
Overview of the existing volumes
```
gluster volume list
```
Remove a volume
```
gluster volume delete firstvol
```

Remove metadata/attributes from the old bricks or simply remove/create the brick sub-dir
```
setattr -x
```

Check attributes:
```
getfattr -d -m'.*' 
```

# TESTING VOLUMES:
	//Install clients
	yum install glusterfs-fuse
	mkdir /mnt/firstvol
	mount -t glusterfs servera:/bricks/brick-a1/brick /mnt/firstvol
	
	//Write files
	touch /mnt/firstvol/file{0..099}
	umount /mnt/firstvol

# VOLUME TYPES:
Gluster can combine bricks into volumes in 3 main ways:
- Distributed (Default): Any file always lives on one brick.
	Allows for a maximum usage of available storage.
	Possible lost of the files stored on a specific brick if it becomes unavailable.
	> Useful when total storage space is more important than availability.
```
gluster volume create VOL_NAME <BRICKS>
```
- Replicated: Bricks are mirrored to each other (A file written to one brick will also be written to one or more other bricks)
		> Offer redundancy and HA, but at the cost of storage capacity (reduced)
```
gluster volume create VOL_NAME replica 2 <BRICKS>
```
- Dispersed: Fragments of the same object are dispersed across different bricks.
	Based on Erasure Coding (EC), which is a method of data protection where data is broken into fragments, expanded and encoded with redundant data pieces, and stored across a set of different locations.
	> RAID 5 or 6 = Protection on the block level <=> DISPERSED VOL = Proection on the file level
	> Dispersed volumes use a number of bricks equal to disperse-data plus redundancy.
	> Supported layouts:
		- Number of bricks(N) = Min Nb of online Bricks (K) + Allowed failed bricks (K) >> N=K+M
		- 6 Bricks for a redundancy level 2 (4+2)
		- 11 Bricks for a redundancy level 3 (8+3)
		- 12 Bricks for a redundancy level 4 (8+4)
```
gluster volume create VOL_NAME diperse-data K redundancy M <BRICKS>
gluster volume create dispesevol disperse-data 4 redundancy 2 brick1 brick2 brick3 brick4 brick5 brick6
```
- Distributed-Replicated volumes: A number of replicated sets is formed equal to the number of bricks in the volume divided by the replica count. Files are then distributed acros these replicated sets.
	> Make sure to specify a number of bricks that is a multiple of the replica count, or a multiple of the dispersed-data + redundancy count.
	> Distributed-replicated volumes are sometimes referred to as X x Y volumes. X=DistributionCount; Y=ReplicaCount
```
gluster volume create distrepvol replica 2 servera:/bricks/brick-a3/brick serverc:/bricks/brick-b3/brick serverd:/bricks/brick-c3/brick servere:/bricks/brick-d3/brick
```
- Distributed-Dispersed volumes: A number of dispersed data is formed equal to the number of bricks in the volume divided by the (disperse-data + redundancy) count. Files are then distributed across these dispersed sets.
	> Distributed-dispersed volumes are referred to as Xx(K+M) volumes. X=DistributionCount; K=disperse-dataCount; M=RedundancyCount.

# SETTING VOLUME OPTIONS:
Volume-specific options can be set. These options range from access control to low-level tuning options for volumes.

Query all options that has been set to a volume
```
gluster volume info mediadata
```

Check all set options
```
gluster volume get mediadata get all
```
Set a volume option
```
gluster volume set mediadata features.read-only on
```

Check the current option's value
```
gluster volume get mediadata features.read-only
```

Reset the default value
```
gluster volume reset mediadata features.read-only
```

> # Options to not forget !
>	- auth.allow *		A comma-separated list of native clients that are allowed to connect to this volume (10.0.0.*)
>	- auth.reject		A list of client that never be allowed to connect to this volume using native client.
>	- nfs.rpc-auth-allow	List of clients granted to connect using NFSv3 client
>	- nfs.rpc-auth-reject
>	- nfs.disable		When set to on, the volume will not be exported over NFSv3
>	- features.read-only	Export the volume in read only for all clients accessing it (on)
>	- server.root-squash	(same as when using the rootsquash option on an NFS export)
>	- user.cifs		Disable/enable SMB sharing
> => When changing access rules, you will sometimes need to stop then start your volume in order for the changes to become active. 

# TIMEOUTS and WARNINGS:

Setting the default SOFT-LIMIT:
```
gluster volume quota <VOLUME> default-soft-limit <SOFTLIMIT-PERCENTAGE>
```

Setting quota soft-timeouts: (To update the usage tables for quotas at specified interval)
```
gluster volume quota <VOLUME> soft-timeout <TIMEOUT>
gluster volume quota graphics soft-timeout 5s
```

Setting quota hard-timeouts:
```
gluster volume quota <VOLUME> hard-timeout <TIMEOUT>
gluster volume quota graphics hard-timeout 1s
```
	
# REPORTING QUOTAS:
Check the quota usage for a volume (current limits and usage..)
```
gluster volume quota <VOLUME> list
```
Volumes can be configured to report quota information
Enable client-side quota reporting using "df"
```
gluster volume set <VOLUME> quota-deem-statfs on
```

# EXTENDING VOLUMES:
Growing a running volumes, by adding bricks to it
```
gluster volume add-brick <VOLUME> replica <NEW-REPLICA-COUNT> BRICK
gluster volume add-brick <VOLUME> replica 2 serverc:/bricks serverd:/bricks
```
Rebalancing volumes after shrinking or extending volumes (Moves files between bricks to match a new distributed layout)
```
gluster volume rebalance <VOLUME> start
gluster volume rebalance <VOLUME> status
```

Throttling reblance operations to avoid negative performance impact on a volume
```
gluster volume set <VOLUME> cluster.rebal-throttle aggressive|normal|lazy
```
SHRINKING VOLUMES: 
Volumes can be shrunk while online by removing one or more bricks

Start the remove command
```
gluster volume remove-brick <VOLUME> serverc:/brick/brick-c1/brick start
```
Monitor the bricks removal progress
```
gluster volume remove-brick <VOLUME> serverc:/brick/brick-c1/brick status
```
Commit your changes, once bricks have finished migrating their data
```
gluster volume remove-brick <VOLUME> serverc:/brick/brick-c1/brick commit
```
Rebalance the volume
```
gluster volume rebalance <VOLUME> start
```
