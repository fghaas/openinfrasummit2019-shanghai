# 10

## Nested virtualization

<!-- Note -->
Nested virtualization is an interesting OpenStack use case.


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


### Live migration
for VMs *not* running nested guests


```ini
[libvirt]
cpu_mode = host-model
```


```ini
[libvirt]
cpu_mode = custom
cpu_model = IvyBridge
```


<!-- .slide: data-background="https://meltdownattack.com/images/meltdown.min.svg" data-background-size="contain" -->


Before Rocky:
```ini
[libvirt]
cpu_mode = custom
cpu_model = IvyBridge
cpu_model_flags = pcid
```
Can’t do nested virtualization (no VMX)


Since Rocky:
```ini
[libvirt]
cpu_mode = custom
cpu_model = IvyBridge
cpu_model_flags = pcid,vmx
```
Can do nested virtualization
