# 5

## Availability Zones

<!-- Note -->
Availability zones in OpenStack have existed for a while, but they
have something of a reputation of mystery — which, at this point,
really is rather undeserved. But you should be aware what they can be
used for, and how exactly they work.


### What are Availability Zones (AZs)?

<!-- Note -->
AZs are meant to divvy up your (or your public cloud provider’s)
_compute environment_ into _failure domains._

That is, by operating AZs a cloud provider makes the following
statement:


> If resources are running in **one** AZ  
> that is affected by an outage,  
> then resources that run in **other** AZs  
> will be unaffected by that **same** outage.

<!-- Note -->
If you are running resources in **one** AZ and there is an outage
affecting that AZ, then resources you choose to run in **other** AZs
will be unaffected by that **same** outage.


<!-- .slide: data-background-image="images/azs.svg" data-background-size="contain" -->
### Intact AZs <!-- .element class="hidden" -->

<!-- Note -->
So here I have an AZ named “Left” running some resources, and an AZ
“Right” running others. So what I can expect is that if one of the AZs
fails, resources in the *other* AZ won’t be affected by that same
problem.


<!-- .slide: data-background-image="images/azs-failed.svg" data-background-size="contain" -->
### Failed AZ <!-- .element class="hidden" -->

<!-- Note -->
Like this: the entirety of my AZ “Left” has failed, but the cloud
service provider is guaranteeing that the resources in the AZ “Right”
are not affected by the same problem.


### What are AZs not?
Host aggregates

<!-- Note -->
AZ are not host aggregates — those are for grouping your compute nodes
by specific characteristics. For example, some of my compute nodes
might have specific GPUs, others may have access to an InfiniBand
fabric.

Those properties can of course overlap, so any compute node can be in
zero or more host aggregrates, but it can only ever be in one AZ.


### What are AZs not?
Cells

<!-- Note -->
AZs are not cells — those are for hierarchical scheduling in OpenStack
environments with very large numbers of compute nodes. AZs are
orthogonal to cells. 


### What are AZs not?
Regions

<!-- Note -->
Regions are separate OpenStack clouds in their own right, normally
unified by a single service catalog, user authentication scheme, and
possibly global image store. 

As such they transcend the compute-only scope that is inherent to AZs,
cells, and host aggregrates. Any region contain one or more AZs, but
any AZ can only ever be in a single region.


### What supports AZs?
Nova

Cinder

Neutron

<!-- Note -->
AZs are supported by Nova (`nova-compute`), Cinder (`cinder-volume`),
and Neutron (the gateway node agents).

There’s a small but important difference between Nova and Cinder on
one hand, and Neutron on the other: in Nova and Cinder, you get to set
exactly one AZ if you’re creating an instance or volume. If no
resources are available in that AZ, that’s a scheduling failure. In
Neutron however, we can define a list of AZs, so if one AZ is not
available for say a router, then Neutron will try the next, and so on.


### AZs are sticky

<!-- Note -->
One neat feature about AZs is that in many cases, you have to specify
it on one resource only, and others will follow suit. For example, if
you

* create an instance in one Nova AZ,
* and it is configured to boot from volume which is created on-the-fly
  from an image,
* and there’s also a Cinder AZ of the same name,

then the Cinder volume will be created in the “correct” AZ (i.e. the
one that matches the instance).

And this comes in particularly handy in combination with the next
feature I’ll talk about.
