# 8

## Qemu Guest Agent

<!-- Note -->
The Qemu Guest Agent (or `qemu-ga`) is a handy libvirt/Qemu/KVM
feature that the hypervisor can use to communicate with the guest.

For you to be able to use it, two prerequisites must be fulfilled.


### hw_qemu_guest_agent=yes <!-- .element class="hidden" -->

```
$ openstack image set \
    --property hw_qemu_guest_agent=yes \
    <image>
```

<!-- Note -->
Firstly, the image you boot off must have the property
`hw_qemu_guest_agent=yes` set. If you set this property, it means that
libvirt will fire up your Nova instance with the virtual serial port
that `qemu-ga` requires to communicate with the instance.


### qemu-guest-agent package <!-- .element class="hidden" -->

```yaml
#cloud-config
packages:
  - qemu-guest-agent
```

<!-- Note -->
Secondly, your instance must install the `qemu-guest-agent` package —
ideally, as you already know by know, via `user-data` and
`cloud-config`.

And once you do that, when Nova wakes one of your VMs from suspend, it
will use its virtual serial port to send the `guest-set-time` command
that `qemu-guest-agent` then processes from within the instance, so
your VM has the correct system time right when it wakes up.

The guest agent also does another thing that’s cool, which is that
when you take a snapshot of an instance, it can automatically freeze
and flush all I/O on all your mounted filesystems, so that the
snapshot you take is one of a clean filesystem.

And speaking of snapshots...
