# 1
## Which OpenStack am I running on?

<!-- Note -->
Suppose you just have access to a virtual machine that you know is
running on OpenStack. You might not even have spun that VM up by
yourself, someone else might have done that for you. All you have is
secure shell access, and you would like to know what OpenStack
platform that VM is running on.

Any takers? Anyone want to offer up a suggestion on how to that?


## `dmidecode`

<!-- Note -->
The easiest way to get to that information is actually this command,
which you might already be quite familiar with.

You’ve probably used `dmidecode` on your laptop, on a baremetal
server, or perhaps even on your Raspberry Pi or OpenWRT router.

It gives you a large amount of information about your system —
including characteristics of your CPU and RAM, and network hardware,
and storage, and peripherals —, and at the very top of its output, it
has this:


## dmidecode output <!-- .element class="hidden" -->

`dmidecode`

```
Handle 0x0100, DMI type 1, 27 bytes
System Information
        Manufacturer: OpenStack Foundation
        Product Name: OpenStack Nova
        Version: 17.0.6
        Serial Number: 452ca901-927a-4baa-9be9-9fa22d36b17c
        UUID: 46AFA4EE-FBA7-49EE-B491-FE7A028D2B88
        Wake-up Type: Power Switch
        SKU Number: Not Specified
        Family: Virtual Machine
```

<!-- Note -->
This information is actually populated by Nova, in conjunction with
Libvirt, and not only does it tell you the Nova release you’re running
on, it also gives you some information about the OpenStack flavor
that’s being deployed (RDO has its own _Product Name_ entry, for
example).

Sometimes this can come in quite handy, for example when you want to
verify what your cloud service provider is telling you about what
OpenStack release they are running in a certain region.
