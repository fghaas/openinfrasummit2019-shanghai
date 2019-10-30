# 2
## Fast instance configuration
with user-data

<!-- Note -->
The second thing I’d like to talk about is a pretty remarkably
under-utilized OpenStack Nova feature, namely Nova user-data. 


## openstack server create <!-- .element class="hidden" -->

```bash
openstack server create \
  --image bionic-server-cloudimg-amd64 \
  --key-name mykey \
  --flavor m1.small \
  --user-data userdata.txt \
  --nic net-id=4f0dcc21-4b6c-47db-b283-591fdb9aa5a7 \
  test0
```

<!-- Note -->
User data is essentially a text file that is made available to the
instance — and this can happen in several ways, it can be injected
into the instance via a magic HTTP URL, or via config-drive, etc. —,
and then that data is picked up by `cloud-init`, running in the
instance operating system on first boot.

We have multiple ways to specify how user data should be loaded into
the instance, but the simplest is to just pass the `--user-data`
option to `openstack server create`, as you can see here.


## frobnication example <!-- .element class="hidden" -->

```sh
#!/bin/sh -e

# Frobnicate a newly booted box

initialize_box
for foo in frobnications; do
  frobnicate_machine $foo || break
done

exit $?
```
# Rejected <!-- .element class="fragment stamp" -->

<!-- Note -->
And *often*, this is what `user-data` looks like. `cloud-init` will
parse any “shebang” (`#!`) at the top of the user-data as a reference
to an interpreter, and if that interpreter is available to the system,
then it will execute the script content using that interpreter.

And most people use a shell script for this purpose. But it _could_
also be, say, a Python or Perl script.

* It’s not a good idea to do this.

Chances are, if you’re writing your user-data in shell, Python, Perl,
or whatever, you’ll be doing a lot of things by hand, that you can
really do much simpler.


## What’s wrong with scripts in user-data?

<!-- Note -->
Now to illustrate what I mean, and **why** using scripts — of any
type: shell, Python, Perl, whatever — in user-data is a bad idea, let
me give you a quick example.


## Consider: upgrading all packages on first boot <!-- .element class="hidden" -->
Consider: upgrading all packages on first boot

<!-- Note -->
Consider this: 

Say you want to update all your installed packages on first
boot. That’s generally a smart and perfectly reasonable thing to do,
because your image may be several months old and may lack critical
patches. And of course you don’t want to have an insecure, unpatched
system from the get-go.


## Multiple package managers <!-- .element class="hidden" -->

Detect distro, then invoke:

`apt-get` <!-- .element class="fragment" -->

`zypper` <!-- .element class="fragment" -->

`yum` <!-- .element class="fragment" -->

`dnf` <!-- .element class="fragment" -->

<!-- Note -->
So now, you’d have to first detect what system you’re running on, and
then depending on that, your shell script would have to invoke

* `apt` or `apt-get` on Debian and Ubuntu,
* `zypper` on SLES and openSUSE,
* `yum` on older Fedora and CentOS releases,
* `dnf` on newer Fedora and CentOS releases,

and so on. It would be messy. And you can do it much more easily.


### Instead: use `cloud-config`

<!-- Note -->
What you *really* want to do is use `cloud-config`. `cloud-config` is
a YAML-based format that `cloud-init` will **also** happily parse for
you, as it does with shell scripts, except you can do things much more
easily. Such as:


## package_update/package_upgrade <!-- .element class="hidden" -->

```yaml
#cloud-config

package_update: true
package_upgrade: true
```

<!-- Note -->
This is is **all** you need to put into your user data in order to get
your system’s installed packages to the latest that are
available. That’s just two lines of YAML, and those will achieve the
desired effect on **any** system that `cloud-init` runs on.

Meaning it’s now `cloud-init` that takes care of the decision of
whether it needs to invoke `apt`, or `zypper`, or `dnf`, or `yum` to
update the package cache and upgrade all installed packages; you no
longer need to take care of that yourself.


## users <!-- .element class="hidden" -->

```yaml
users:
- default
- name: foobar
  gecos: "Fred Otto Oscar Bar"
  groups: users,adm
  lock-passwd: false
  passwd: $6$rounds=4096$J86aZz0Q$To16RGzWJku0
  shell: /bin/bash
  sudo: "ALL=(ALL) NOPASSWD:ALL"
```

<!-- Note -->
You can also define users, including their group membership, preferred
shell, `sudo` privileges, and you can even preseed their password.


## ssh_pwauth <!-- .element class="hidden" -->

```yaml
ssh_pwauth: true
```

<!-- Note -->
You might want to enable users to log into cloud instances using their
username and password, as opposed to their public SSH key. 

That’s a single boolean that you need to set, and you will no longer
need to worry about what the `sshd` daemon is named on the target
platform, or what its configuration file is, or what’s the right
syntax to reload the daemon, or anything else.


## write_files <!-- .element class="hidden" -->

```yaml
write_files:
- path: /etc/hosts
  permissions: '0644'
  content: |
    127.0.0.1 localhost
    ::1       ip6-localhost ip6-loopback
    fe00::0   ip6-localnet
    ff00::0   ip6-mcastprefix
    ff02::1   ip6-allnodes
    ff02::2   ip6-allrouters

    192.168.122.100 deploy.example.com deploy
    192.168.122.111 alice.example.com alice
```

<!-- Note -->
You might want to write out arbitrary files, and you can do that, too,
including setting their ownership and permission bits.


## packages <!-- .element class="hidden" -->

```yaml
packages:
  - ansible
  - git
```

<!-- Note -->
You can install additional packages not included in the base image, by
simply providing a `packages` list.


## runcmd <!-- .element class="hidden" -->

```yaml
runcmd:
  - >
    sudo -i -u training
    ansible-pull -v -i hosts
    -U https://github.com/hastexo/academy-ansible
    -o site.yml
```

<!-- Note -->
And even if you *do* run into something that requires a script
because `cloud-config` doesn’t support it natively, you can do that
too, with the `runcmd` list that specifies a list of commands to run
at the end of the initial boot sequence.

So, bottom line: user-data is something you should totally use, but
use it right. Use `cloud-config` and make full use of `cloud-init`’s
features, don’t write your own scripts.
