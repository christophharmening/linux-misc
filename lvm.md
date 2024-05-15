# LVM

## Extend a lvm logical volume

Build in a new hdd and identify it with
``
lsblk
``

Create a partition table with gdisk
``
gdisk /dev/sdb
`` 

Identify you volume group
simple
``
vgs
``
or detailed
``
vgdidisplay
``

Expand you volume group
``
vgextend vg-system /dev/sdb1
``

Identify your logical volume
simple
``
lvs
``
or detailed
``
lvdisplay
``

Expand your logical volume to maximum
`` lvextend -l +100FREE /dev/mapper/vg-system--lv-home ``
