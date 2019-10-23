# 3

## Fast instance spinup with Ceph

<!-- Note -->
I want to talk about the importance of fast instance spin-up for a
moment. Frequently, people would think that it doesn’t make much of a
difference if we can spin up a Nova instance in, say, one or ten
minutes. And that may well be true for some use cases, but you should
be aware that there are others where instance spin-up speed is
absolutely a factor. This is generally true under any circumstances
where you’re aiming for maximum scalability and elasticity.


### Ceph
One storage backend to rule them all

<!-- Note -->
And I want to talk about one specific storage technology, which is
particularly well suited for that purpose, and that’s Ceph.


### Ceph
... for Glance/Cinder/Nova

<!-- Note -->
Now why is that so? It’s because we can use one and the same Ceph
storage cluster for Glance image storage, Cinder volumes, and Nova
ephemeral storage.


### RBD
Snapshots and Clones

<!-- Note --> RBD is the **RADOS Block Device,** which basically means that
there is a device that behaves just like a regular block device, but
underneath does some magic that translates all block I/O into RADOS
object access. Like many other block device types, RBD is capable of
creating

* _snapshots,_ that is a consistent read-only view of the block device
  at any arbitrary point-in-time;
* _clones,_ that is an efficient, writable exception store to a
  snapshot (or less technically speaking, a writable snapshot).

And in OpenStack, we can use all those left, right and center.


### Boot from image
Glance RBD snapshot -> Nova RBD clone


### Boot from volume
Glance RBD snapshot -> Cinder RBD clone


### Image from volume
Cinder RBD snapshot -> Glance RBD clone


### Volume snapshot
Cinder RBD volume -> Cinder RBD snapshot


### Volume from image
Glance RBD snapshot -> Cinder RBD clone


### Everything in Ceph
No image streaming necessary


### Everything in Ceph
All storage operations happen in backend


### Important:
`raw` Glance images only
