# SETTING POSIX ACLs: # 
Setting ACLs for fine-grained access control to files and directories

Enable ACLs by adding ACL as a mount option:
```
vim /etc/fstab
servera:/groupdata /mnt/groupdata glusterfs _netdev,acl 0 0
```

Apply new ACL, to recursively give the user lisa R/W access to all files in the /springfield-library:
```
setfacl -R -m u:lisa:rwX /springfield-library
```

Ensure that lisa will get R/W permissions on any new files and directories created in springfield-library:
```
setfacl -R -m d:u:lisa:rwX /springfield-library
```

Remove ACL:
```
setfacl -R -x 
```

# Apply a shared ACLs / Shared directory.
Mounted directory should be owned nby "accountants" group:
```
chgrp accountants /mnt/finance/profits
```	

Any new files/directories should also be owned by the "accountants" group
Any other users should have no access
```
chmod 2770 /mnt/finance/profits
```
	
Allow full access to to members of admins group and ensure that they will got access to any new ones
```
setfacl -R -m g:admins:rwX /mnt/groupdata/admindocs
setfacl -R -m d:g:admins:rwX /mnt/groupdata/admindocs
```

# QUOTAS:
Enable directory quotas or volume quotas
```
	gluster volume quota <VOLUME> enable
```
> Quotas are not set for the entire volume by default.
Quotas are set as a :
	- hard-limit - the maximum allowed size of all files and directories under a directory
	- soft-limit - a percentage of the hard limit (if soft-limit are exceeded, entries will be placed in the log files for that bricks that make up the volume)

Set quotas as a hard-limit on a directory
```
	gluster volume quota <VOLUME> limit-usage <PATH-ON-VOLUME> <SIZE> <SOFTLIMIT-PERCENTAGE>
	gluster volume quota graphics limit-usage /raw 1GB 50%
```
