# 9

## Stack snapshot and rollback

<!-- Note -->
We can apply a snapshot operation not just to a single instance, but
to a whole Heat stack.


### openstack stack snapshot create <!-- .element class="hidden" -->

```bash
openstack stack snapshot create \
  --name mysnap \
  mystack
```
Creates a snapshot of an entire Heat stack.

<!-- Note -->
With `openstack stack snapshot create`, we can create a point-in-time
view of an entire Heat stack. Now, you would be forgiven to think that
what happens in the background here is that we take

* a snapshot of every running Nova _instance,_ and
* a snapshot of every Cinder _volume_ in the stack.

How it _actually_ works though, is a slightly different story.


### Stack snapshots
What really happens

<!-- Note -->
When initiating a stack snapshot, `heat-engine` does interact with
Nova to create a snapshot image from all the Nova instances
(i.e. `OS::Nova::Server` resources) in the stack. (How long this takes
is primarily a matter of what backends are in use for Nova ephemeral
storage and Glance image storage.)

What `heat-engine` does with respect to Cinder volumes
(`OS::Cinder::Volume` resources) is actually something _other_ than
create a volume _snapshot:_ in reality, it creates a volume _backup_
using the `cinder-backup` service. If your cloud doesn’t run
`cinder-backup`, then no snapshots for you. 

The thinking behind this is that if you are ever to *delete* a stack
(and delete its volumes in the process), then the volume *snapshots*
go away as well. You normally don’t want that, instead you want your
volume contents to be preserved for posterity in some way. Backups do
that for you.
