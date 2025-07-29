---
id: eh53s630gs9xdxtsjdkfvh1
title: Rook Ceph Cheatsheet
desc: ""
updated: 1680419665921
created: 1680416917727
---

## Fetch Rook Ceph Dashboard Credentials

Username: `admin`

```bash
kubectl get secrets rook-ceph-dashboard-password -o jsonpath="{.data.password}"  | base64 -d
```

## Create rook toolbox

```bash
kubectl create -f https://raw.githubusercontent.com/rook/rook/master/deploy/examples/toolbox.yaml
```

## Purge Rook Ceph Node

All the commands require root privileges.

Remove `/var/lib/rook`

```bash
rm -rf /var/lib/rook
```

Purge disk

```bash
# purge disk for sdX, sdY, sdZ
for i in sdX sdY sdZ; do
DISK="/dev/${i}"
# Zap the disk to a fresh, usable state (zap-all is important, b/c MBR has to be clean)
sgdisk --zap-all $DISK

# Wipe a large portion of the beginning of the disk to remove more LVM metadata that may be present
dd if=/dev/zero of="$DISK" bs=1M count=100 oflag=direct,dsync

# SSDs may be better cleaned with blkdiscard instead of dd
blkdiscard $DISK

# Inform the OS of partition table changes
partprobe $DISK
done
```

Purge Device Mapper

```bash
# This command hangs on some systems: with caution, 'dmsetup remove_all --force' can be used
ls /dev/mapper/ceph-* | xargs -I% -- dmsetup remove %

# ceph-volume setup can leave ceph-<UUID> directories in /dev and /dev/mapper (unnecessary clutter)
rm -rf /dev/ceph-*
rm -rf /dev/mapper/ceph--*
```
