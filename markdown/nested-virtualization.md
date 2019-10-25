# 10

## Nested virtualization

<!-- Note -->
And finally: nested virtualization. This is an interesting OpenStack
feature.

Most people usually don’t need it, but if you do, it comes in quite
handy.


### Running KVM in KVM

<!-- Note -->
You may want the ability to run KVM inside your Nova virtual machines,
that is to say run VMs _with hardware acceleration_ within your Nova
instances that are themselves running in KVM. (In KVM parlance, this
is known as “Nested Guests”.)


### No suspend
while running nested guests

<!-- Note -->
When a Nova instance runs nested guests, you can’t suspend it. That
is, you can, but when the instance resumes, it’ll kernel-panic.


### No live migration
while running nested guests

<!-- Note -->
Under the covers, live migration and suspension are really one and the
same. That is, what suspend writes to a file, live migration writes to
a socket file descriptor. So, same thing: the VM will panic after live
migration. This is being worked on by the KVM developers, but
currently is an inherent KVM limitation. If a VM merely has nested
virtualization _enabled_ but doesn’t run any VMs, it’ll be fine.

Now there’s an interesting consideration here, and this one is
primarily for those of you *operating* OpenStack clouds, but even if
you’re only *using* one, it’s still potentially interesting.


### cpu_mode = host-model <!-- .element class="hidden" -->

```ini
[libvirt]
cpu_mode = host-model
```

<!-- Note -->
By default, Nova/libvirt configures all your guest VMs to use the
“host model” CPU, meaning it will configure all running instances that
are scheduled to a particular host with a set of CPU feature flags
that most closely match the feature flags of the host itself.

This sometimes creates issues with live migration. If you live-migrate
from a host with a newer CPU to one with an older CPU, then the VM
can’t find the feature flags it needs on the target machine, and the
migration just fails. This is why, frequently, operators decide to
configure their Nova compute nodes like this:


### cpu_model = IvyBridge <!-- .element class="hidden" -->

```ini
[libvirt]
cpu_mode = custom
cpu_model = IvyBridge
```

<!-- Note -->
So what happens here is you select a “custom” CPU model, and that
model is a processor that corresponds to the oldest CPU in your
hypervisor inventory.

When you do that (or have done that), you may be in for a rude
surprise.


<!-- .slide: data-background="images/meltdown.min.svg" data-background-size="contain" -->
### Meltdown <!-- .element class="hidden" -->

<!-- Note -->
The Meltdown attack has caused a whole flurry of mitigation patches to
the Linux kernel, many of which have a *significant* performance
impact on virtualization workloads **except** if the VM has access
to...


### PCID
Process Context ID

<!-- Note -->
... the Process Context ID (PCID) CPU flag.

And some popular `cpu_model` options, such as the aforementioned
`IvyBridge`, *do not pass this flag in by default.*

This means you now need to override this in your Nova configuration,
so that you *do* get this flag passed in, otherwise you’re taking a
really bad performance hit.


### cpu_model_extra_flags = pcid <!-- .element class="hidden" -->

Before Rocky:
```ini
[libvirt]
cpu_mode = custom
cpu_model = IvyBridge
cpu_model_extra_flags = pcid
```
Can’t do nested virtualization (no VMX) <!-- .element class="fragment" -->

<!-- Note -->
You do this by setting the `cpu_model_extra_flags` option to include the
`pcid` CPU flag.

Now here’s a limitation that comes into play if you are using nested
virtualization: in OpenStack releases up to and including Queens, the
`vmx` flag is *not* an available `cpu_model_extra_flags` option.

* And that means that you don’t get nested virtualization in `cpu_mode
  = custom` with an old `cpu_model`.
  
However:


### cpu_model_extra_flags = pcid,vmx <!-- .element class="hidden" -->

Since Rocky:
```ini
[libvirt]
cpu_mode = custom
cpu_model = IvyBridge
cpu_model_extra_flags = pcid,vmx
```
Can do nested virtualization

<!-- Note -->
Since Rocky, you can set this option to include *both* the `pcid` and
the `vmx` flag, and that means that you can mitigate the impact of the
Meltdown patches, and at the same time also run nested virtualization.

Which is nice.
