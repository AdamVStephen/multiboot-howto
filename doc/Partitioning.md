# Partitioning

Some installers require a GPT partition table, though it is possible to make LVM volume groups directly on a raw disk device.

## Overallocation for Flexibility

Taking a 1TB disk as the example, we overallocate in order to allow the ext4 file systems to be grown/shrunk.
This is also called thin provisioning and can be done as follows

## Demonstration of Principle

Partition the disk 

```
parted -a optimal /dev/nvme0n1
unit MiB
mkpart 1 1 256
name 1 boot
set 1 legacy_boot on
mkpart 2 256 100%
set 2 lvm on
```

Now create a PV on /dev/nvme0n1p2, a volume group, a thin pool, and two over allocated logical volumes.
We then create `ext4` filesystems on each and mount the file systems.

```
pvcreate /dev/nvme0n1p2
vgcreate vg0 /dev/nvme0n1p2
lvcreate -L 10G --thinpool vg0_thinpool
lvcreate -V 10G --thin -n vg0.lv0
lvcreate -V 10G --thin -n vg0.lv1
mkfs.ext4 /dev/vg0_thinpool/vg0.lv0
mkfs.ext4 /dev/vg0_thinpool/vg0.lv1
mkdir -p /mnt/thinvol{0,1}
mount /dev/vg0_thinpool/vg0.lv0 /mnt/thinvol0
mount /dev/vg0_thinpool/vg0.lv1 /mnt/thinvol1
```
