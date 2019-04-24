# GlusterFS Cheat Sheet

Red Hat Gluster Storage is based on the work of the GlusterFS project to provide enterprise class storage solutions built on top of commodity hardware.

Red Hat Gluster Storage combines file and object storage with a scaled-out architecture.

#### FEATURES:
- Scale-Up/Scale-Out:  pool capacity, redundancy, and availability
- Decentralized: does not rely on a central metadata server to locate files. 
- High Availability: synchronous mirroring across multiple servers. 
- No Application Rewrites: provides POSIX-compliant file systems to clients
- Commodity Hardware 
- Bare Metal and Cloud 

#### TERMINOLOGY:
- Trusted Storage Pool: When a Red Hat Gluster Storage server first starts, it belongs to a Trusted Storage Pool with only itself as a member. Additional servers can be added later to form a larger pool.
- Node: A server participating in a Trusted Storage Pool or cluster
- Brick: Gluster Storage allows multiple file systems (called Bricks) to be combined into one larger file system called a Volume.
	A brick can only be a member of a single volume.
	Brick = storageserver1.example.com:/exports/myfirstbrick
- Metadata: Metadata is data about one or more other pieces of data (file names, last modification time, owner, permissions,..)
- Volume: A volume is a file system presented to clients. Consists of one or more bricks on which data is stored
- Elastic Hash Algorithm: Each server in a pool has the intelligence to locate any piece of data without having to query an index or another server = hashing the file name of the requested data to locate the required brick in the target volume.
