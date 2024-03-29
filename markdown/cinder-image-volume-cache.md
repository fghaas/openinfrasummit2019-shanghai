# 6

## The Cinder image-volume cache

<!-- Note -->
This feature is a bit obscure because it is especially useful in a
very specific scenario.

Consider this: as you already know, both Nova and Cinder support
Availability Zones, and they do play rather nicely with one
another. So you could have multiple physical locations, as shown here:


### Cinder and Glance in multi-AZ setup <!-- .element class="hidden" -->
<!-- .slide: data-background="images/cinder-image-volume-cache.svg" data-background-size="contain" data-timing="60" -->

<!-- Note -->
Here we’ve got a single OpenStack region, with three data centers
representing three AZs. The AZ Center is the one that has compute
infrastructure plus our control nodes, and the AZs Left and Right have
just compute infrastructure.

And what’s more, every AZ uses a different storage backend: in this
example, the Center AZ uses Ceph, the Left AZ uses EMC² storage, and
the Right AZ uses NetApp storage.

Now there’s one thing in this diagram that *doesn’t* know about AZs,
and that’s Glance. Indeed, we currently have no way of dispersing
Glance across multiple locations, short of having multiple Glance API
endpoints and defining specific regions for them.

So conventionally, if an instance is fired up in one of the remote
locations, it would do one of two things:

* _Either_ it would stream the image down to the compute node _once
  for each compute node._ That’s a pretty bad waste of bandwidth, and
  also it means really slow instance spin-up.
* _Or_ it would assume that it has permanent access to the original
  image, and create a shallow copy-on-write clone from it (this is
  normally what happens if our Glance backend is Ceph). That’s very
  easy on bandwidth, but terrible in terms of latency.

So what can we do?


### image_volume_cache_enabled = True <!-- .element class="hidden" -->
```ini
image_volume_cache_enabled = True
```
(works globally, or per backend)

<!-- Note -->
We can add a single setting to our `cinder.conf` that toggles the
image-volume cache on and off. When it’s on, then _every time an image
is used for the first time,_ it is streamed down to the Cinder
backend, and a copy is created there. Every subsequent use of that
image then doesn’t need to be fetched from the primary Glance
location, but can simply be served from the cache.


### Image-volume cache limits <!-- .element class="hidden" -->
```ini
image_volume_cache_enabled = True
image_volume_cache_max_size_gb = 200
image_volume_cache_max_count = 50
```

<!-- Note -->
We can also limit the maximum size of an image to be cached in the
local Cinder store — this is the maximum size _per image,_ not the
total image size – and/or limit the maximum number of images to be
cached at any given time. 
