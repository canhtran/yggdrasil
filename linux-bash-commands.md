# Linux bash commands

Partition and disks

```bash
# list all block device
$ lsblk
# list hard disks
$ fdisk -l
# format partition
$ mkfs.ext3 /dev/xvdf
# mount partition
$ mount /dev/xvdf /data
```

