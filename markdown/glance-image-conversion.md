# 4

## Image conversion in Glance

<!-- Note -->
Suppose you have an image, in Glance, and it’s in the wrong image
format. For example, you may have uploaded an image in the `qcow2`
format, but because you’ve since realized that you’d be in better
shape with the `raw` format, you’d like to convert it.

Normally, that means you need to download the image, convert it
locally (with `qemu-img convert`), and re-upload it.

There’s also an extremely arcane taskflow syntax to do this in the
Glance backend, but I’m about to show you a super easy way.


```bash
openstack volume create \
  --image=<image> \
  --size=<size> \
  <volume>
```
... gives you a volume (always raw) <!-- .element class="fragment" -->

<!-- Note -->
`openstack volume create`, of course, creates a new volume. You specify
the desired size, and by passing in `--image=<image ID>`, you
initialize the volume with the contents of the image.

* And importantly, what happens here is that the Glance image
  automatically gets converted to the `raw` format as it being piped
  into the volume.


```bash
openstack image create \
  --volume=<volume> \
  <name>
```
... preserves the volume format <!-- .element class="fragment" -->

<!-- Note -->
And now with `openstack image create`, you create a new volume. This
time, by passing in `--volume=<volume>`, you tell Glance to create an
image populated with the volume’s content. 

* That image always retains the format of its source volume, so what
  you end up with is effectively a copy of your original image, albeit
  in the `raw` format.

So that’s a quick and easy way to convert images to the `raw` format,
which doesn’t require tedious downloading and uploading.
