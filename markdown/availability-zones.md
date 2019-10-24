# 5

## Availability Zones

<!-- Note --> 
Availability zones in OpenStack have existed for a while, but
they have something of a bad reputation — which, at this point, really
is rather undeserved. But you should be aware what they can be used
for, and how exactly they work.


### What are AZs?

<!-- Note -->
AZs are meant to divvy up your (or your public cloud provider’s)
_compute environment_ into _failure domains._ That is, by operating
AZs a cloud provider makes the following statement: “If you are
running resources in one AZ and there is an outage
affecting that AZ, then resources you choose to run in _other_ AZs
will be unaffected by that same outage.”


### What are AZs not?
Host aggregates

<!-- Note -->
AZ are not host aggregates — those are for grouping your compute nodes
by specific characteristics. Any compute node can be in zero or more
host aggregrates, but it can only ever be in one AZ.


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
possibly global image store. As such they transcend the compute-only
scope that is inherent to AZs, cells, and host aggregrates. Any region
contain zero or more AZs, but any AZ can only ever be in a single
region.


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


### AZs follow each other
