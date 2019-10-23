# 8

## Suspend and Resume

<!-- Note -->
Being able to suspend and resume instances — that is, virtual machines
— and even arbitrarily complex distributed environments as a whole —
is an extremely useful OpenStack feature. But it’s important to
understand what these actions do.


### `openstack server suspend`
... doesn’t suspend your server.

<!-- Note --> 
Most people would think that `openstack server suspend` (or its
deprecated equivalent, `nova suspend`) essentially just send a `STOP`
signal to a running hypervisor process, and then the corresponding
`resume` command would send a `CONT` signal. Indeed, that’s what
`virsh suspend` does in libvirt. **That is not what it does.**

Instead, `suspend` in Nova-speak corresponds to what libvirt calls a
“managed save” operation: the content of the virtual machine’s memory
is written to a file (called a “savefile”), and then the KVM instance
is shut down. When it starts back up, libvirt detects that there is a
savefile, and immediately after bringing up the instance in a paused
state, populates its memory from the savefile.


### `openstack server suspend`
... is (almost) free.

<!-- Note -->
Now this means that while the instance is “suspended”, it consumes
**zero** RAM, and **zero** CPU cycles. Since RAM-hours and CPU-hours
are by far the most expensive thing to pay for in a public-cloud VM,
that means that your instance will rack up almost no cost while it’s
suspended.

I say *almost* because your cloud service provider will still charge
you for things allocated disk space and publicly routable IP
addresses, because those of course can’t be reclaimed while your
machine is dormant, but it’s going to amount to a tiny fraction of the
cost of a VM running throughout.


### `openstack server suspend`
... is great for test environments.


### `openstack stack suspend`
... is even better.
