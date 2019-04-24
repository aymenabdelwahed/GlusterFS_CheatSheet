## MANAGE BRICKS/VOLUMES
In order to start building volumes, it will first be necessary to create bricks.
> A Brick is an XFS file system (with 512-byte inodes) mounted on one of the storage servers !
> It's recommended to create bricks on top of thinly provisioned logical volumes.

### CREATE THIN LOGICAL VOLUMES:
Check existing volume groups
```
vgs
```
Create a thin-provisioned LogicalVolume: (An LVM thin pool is created like a logical volume, but with flags set to mark it as a pool of available storage for thin volumes)
```
lvcreate -L 10G -T vg_bricks/examplepool
```
or
```
lvcreate --thin --size 10G vg_bricks/thinpool
lvs
```
Create a Thin Volume inside the newly created LogicalVolume (-V to specify a virtual size)
Thin volumes doesn't have a physical size but a virtual Size (-V)
```
lvcreate -V 2G -T vg_bricks/examplepool -n brick-a1
lvcreate -V 2G -T vg_bricks/examplepool -n brick-b1
lvs	
lvdisplay
```
Format with a filesystem of larger inodes' size to store metadata (Extended Attributes are inodes; inodes are by default 256Bytes which is insufficient for metadata)
```
mkfs.xfs -i size=512 /dev/vg_bricks/brick-a1
mkfs.xfs -i size=512 /dev/vg_bricks/brick-b1
```
> Gluster Storage stores metadata in the extended attributes area of the inodes

### CREATE BRICKS:
It's recommended that bricks be given a descriptive mount point to easily identify them (bricks/brick1) with a unique mount point within the storage pool

Devices used for bricks should be LVM based (LVM speeds up the snapshots mgmt, causing minimal performance impacts on the LVM backend system while those are being created)
Create the mountpoint (Must be unique in the trusted storage pool)
```
mkdir -p /bricks/brick-a1
mkdir -p /bricks/brick-b1
```
Create an fstab entry
```
echo "/dev/vg_bricks/brick-a1 /bricks/brick-a1 xfs defaults 1 2" >> /etc/fstab
echo "/dev/vg_bricks/brick-b1 /bricks/brick-b1 xfs defaults 1 2" >> /etc/fstab
```
Mount the vol
```
mount /bricks/thinvol
```
Create a subdirectory on the new filesystem to store the brick. (This will prevent the accidental storage of data on the root filesystem when the brick is not mounted, which can lead to data loss)
```
mkdir /bricks/thinvol/brick
```
Set SELinux context on the new brick directory to allow access:
```
semanage fcontext -a -t glusterd_brick_t /bricks/brick-a1/brick
restorecon -Rv /bricks/thinvol/brick
```
