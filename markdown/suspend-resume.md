# 7

## Suspend and Resume

<!-- Note -->
Being able to suspend and resume instances — that is, virtual machines
— and even arbitrarily complex distributed environments as a whole —
is an extremely useful OpenStack feature. But it’s important to
understand what these actions do.


### `openstack server suspend`
... doesn’t suspend your server. <!-- .element class="fragment" -->

It “managed-saves” your server. <!-- .element class="fragment" -->

<!-- Note --> 
Most people would think that `openstack server suspend` (or its
deprecated equivalent, `nova suspend`) essentially just send a `STOP`
signal to a running hypervisor process, and then the corresponding
`resume` command would send a `CONT` signal. Indeed, that’s what
`virsh suspend` does in libvirt. 

* **That is not what `openstack server suspend` does.**

* Instead, `suspend` in Nova-speak corresponds to what libvirt calls a
  “managed save” operation: the content of the virtual machine’s
  memory is written to a file (called a “savefile”), and then the KVM
  instance is shut down. When it starts back up, libvirt detects that
  there is a savefile, and immediately after bringing up the instance
  in a paused state, populates its memory from the savefile.


### `openstack server suspend`
... is (almost) free.

<!-- Note -->
Now this means that while the instance is “suspended” (managed-saved),
it’s effectively shut off.

That also means it consumes **zero** RAM, and **zero** CPU
cycles. Since RAM-hours and CPU-hours are by far the most expensive
thing to pay for in a public-cloud VM, that means that your instance
will rack up almost no cost while it’s suspended.

I say *almost* because your cloud service provider will still charge
you for things allocated disk space and publicly routable IP
addresses, because those of course can’t be reclaimed while your
machine is dormant, but it’s going to amount to a tiny fraction of the
cost of a VM running throughout.


### `openstack server suspend`
... is great for test environments.

<!-- Note -->
This makes the `suspend`/`resume` approach extremely useful for test
environments that you don’t use all the time.

But what’s even more useful is...


### `openstack stack suspend`
... is even better.

<!-- Note -->
... the fact that you can use the `suspend`/`resume` approach on whole
Heat **stacks,** where it then applies to *all* Nova instances in a
stack, even if you use nested stacks (that is to stay, a parent stack
with multiple children, possibly even nested several layers deep).

Now, if you’re using suspend/resume, you’ll be particularly interested
in the next feature I’m talking about. You see, a common issue when
resuming is that your VM’s system clock may be way out of
whack. Perhaps your application doesn’t care about that, but *if* it
does, here’s what will come in very handy.
