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

Now why is that so? It’s because we can use one and the same Ceph
storage cluster for Glance image storage, Cinder volumes, and Nova
ephemeral storage.


### RBD
Snapshots and Clones

<!-- Note -->
What we use here, specifically, is RBD.

RBD is the **RADOS Block Device,** which basically means that there is
a device that behaves just like a regular block device, but underneath
does some magic that translates all block I/O into RADOS object
access. Like many other block device types, RBD is capable of creating

* _snapshots,_ that is a consistent read-only view of the block device
  at any arbitrary point-in-time;
* _clones,_ that is an efficient, writable exception store to a
  snapshot (or less technically speaking, a writable snapshot).

And in OpenStack, we can use all those left, right and center.


### Boot from image
Glance RBD snapshot → Nova RBD clone

<!-- Note -->
If you boot your instance off of a Ceph-backed Glance image — which
means it’s an RBD —, and your Nova libvirt driver is *also* configured
to use Ceph, then that means there needs to be no streaming the Glance
image down to your nova-compute host (which is a process that can take
many minutes for large images).

Instead, Nova creates an RBD **clone** on your behalf (in the backend,
no data shuffling required), and the instance boots in seconds.


### Boot from volume
Glance RBD snapshot → Cinder RBD clone

<!-- Note -->
You can also use this same approach for boot-from-volume. It’s
essentially exactly the same process, except the clone being created
is now a persistent volume that lives in Cinder. But just as with
ephemeral boot from Cinder, the snapshot creation is practically
instantaneous, and your VM boots in mere seconds.


### Image from volume
Cinder RBD snapshot → Glance RBD clone

<!-- Note -->
Of course you can also use the reverse approach, which is to populate
an image from a volume. To that end, Ceph creates a point-in-time
frozen view of your volume (a *snapshot*), and then populates an image
with that data.


### Volume snapshot
Cinder RBD volume → Cinder RBD snapshot

<!-- Note -->
And of course speaking of snapshots, we can also apply the same
principle for volume snapshots.

The Cinder snapshot, in this case, translates directly into an RBD
snapshot.

This is all pretty straightforward, but it’s also cool because it
makes Cinder snapshots really quite fast when your backend is Ceph
driven.


### Important:
`raw` Glance images only

<!-- Note -->
Now there is one very important consideration that applies here: all
of this cool snapshot and cloning magic, when it comes to Glance
images stored in Ceph, is functional *only* when your RBD-backed
Glance image uses the `raw` format.

When you think about it, that makes a lot of sense: Ceph RBD does all
sorts of cool things — thin provisioning, snapshots, resizing, whatnot
— natively in the backend. Qcow2, in contrast, does it all within its
file format. So if you store Qcow2 in RBD, you’re kind of doing it all
twice, which isn’t very reasonable.

But **most** vendor-provided cloud images come in Qcow2 by default. So
if I have an image in Qcow2 and I want to really have it in the `raw`
format, what do I do?
