### TROUBLESHOOTING:
Troubleshooting native client mounts
```
tail -f /var/log/glusterfs/mnt-<VOLNAME>.log
```
### Managing selft-healing:
For replicated volumes, when a brick has been offline, self-healing is required to resync all replicas. This is handled by self-healing daemon "glusterfshd" running on all nodes with a replicated brick.
Self-healing daemon will identify files that possibly need healing, and then sync the files between the replicated bricks:
View the list of files that are currently need healing:
```
gluster volume heal <VOLUME> info
```
Show the number of files healed, per brick, per pass
```
gluster volume heal <VOLUME> statistics
```
triggering a self-heal
```
gluster volume heal <VOLUME> heal full
```
Without a full option a normal heal is performed
Check split-brain files:
```
gluster volume heal <VOLUME> info split-brain
```

Replacing a defective brick in a distributed volume:
> A brick can be replaced by adding a new brick, and then removing the old brick... > The rebalance operation triggered by the remove of the old brick will automatically move all files to the new brick..

Replcaing a defective brick in a replicated volume:
> Replace bricks= Remove the old brick immediately and add the new brick, triggering a self-heal operation
```
gluster volume replace-brick <VOLUME> OLD_BRICK NEW_BRICK commit force
```
