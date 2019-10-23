# 4

## Image conversion in Glance

<!-- Note -->
Suppose you have an image, in Glance, and it’s in the wrong image
format. For example, you may have uploaded an image in the `qcow2`
format, but because you’ve since realized that you’d be in better
shape with the `raw` format, you’d like to convert it.

Normally, that means you need to download the image, and re-upload
it. There’s also an extremely arcane taskflow syntax to do this in the
Glance backend, but I’m about to show you a super easy way.


```bash
openstack volume create \
  --image=<image>
  --size=<size>
  <volume>
```
... gives you a volume (always `raw`)

<!-- Note -->
By passing in `--image=<image ID>`, you initialize the volume with the
contents of the image.


```bash
openstack image create \
  --volume=<volume>
  <name>
```
... preserves the volume format

<!-- Note -->
By passing in --volume=<volume>, you tell Glance to create an image
populated with the volume’s content. That image always retains the
format of its source volume.
