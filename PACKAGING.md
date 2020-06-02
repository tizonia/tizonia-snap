Tizonia's Snap Development
==========================

# Overview

According to the Snapcraft statistics of the Tizonia Snap, this package is
actively used by around 2500 people. So lots of people would be glad to see
this Snap package frequently updated and in sync with the releases in the main
application repository (https://github.com/tizonia/tizonia-openmax-il).

What follows is a list of resources and a guide to the Tizonia Snap
package. This is the main resource for people wanting to help with the Snap
packaging activities of the Tizonia project.

# First steps

Tizonia is a hybrid application that conflates code written in C, C++ and
Python. While packaging Tizonia as a Snap (or in any other format for that
matter) it is important to keep this in mind.

Another important detail to know about is that Tizonia has recently added a
Meson build system in the main repository. Note that the Meson build system
co-exists with the Autotools build system in the main repo. The idea is to
deprecate Autotools at some point in the future. So the recommended build
system to use while packaging Tizonia is the Meson build system.

# Snap Build Workflow

## Pre-requisites (Ubuntu 18.04 assumed)
- Make sure `snapd` is installed
  - `sudo apt install snapd`
- Remove any old version of snapcraft (in case it was previously installed via
  apt-get)
  - `sudo apt purge snapcraft`
- Install the latest version of `snapcraft` via the snap system
  - `sudo snap install snapcraft --classic`
- Install the latest version of `multipass`
  - `sudo snap install multipass --classic`

## Building the Snap inside a Multipass instance
`snapcraft` nowdays uses [Multipass](https://multipass.run/docs) VMs to build
the snap packages.

### Gotcha #1: /root partition with limited space
Multipass VMs are stored in the 'vault' directory which lives under
`/var/snap/multipass/common`. If you have a /root partition with limited space
(like I do), Multipass can fill up the partition. Unfortunately, at the time of
writing, Multipass does not provide a configuration option to customize the
location of VM files in the system. And it does not warn you that your
partition will be filled!. See the following links for information on possible
workarounds to this problem.:
- https://github.com/canonical/multipass/issues/1215
- https://forum.snapcraft.io/t/moved-var-lib-snapd-into-home-snapd-and-symlicked-back-snaps-fail-to-start/15272/2

### Gotcha #2: Tizonia requires lots of RAM during the build process
Tizonia requires lots of RAM during the build process. By default, Multipass
creates VMs with 2GB or RAM. This won't cut it while building Tizonia and the
compilation will fail. Use the `SNAPCRAFT_BUILD_ENVIRONMENT_MEMORY` environment
variable to instantiate Multipass VMs with at least 8GB of RAM.

E.g.: from the top of the `tizonia-snap` repo
- `SNAPCRAFT_BUILD_ENVIRONMENT_MEMORY=16G snapcraft --debug`

(NOTE: If you have already issued an `snapcraft --debug` with the default RAM,
adding the `SNAPCRAFT_BUILD_ENVIRONMENT_MEMORY` environment variable afterwards
will not work as the VM memory cannot be modified at that point. You will need
to explicitly destroy the existing VM by calling `snapcraft clean`, and then
start over).

While at it, if you have enough CPUs on your machine, you can also increase the
number of CPUs that Multipass will allocate on the VM (the default is 2
CPUs). To make use of the additional CPUs, you need to modify the the number of
parallel jobs that ninja will use during the build. This value is 1 in the
snapcraft.yaml, to avoid running out of memory.

E.g.:
```
ninja -j1 -C build --> ninja -j8 -C build

```

But REMEMBER: The more CPUs used in parallel by ninja, the more RAM required
for the overall build process.


E.g.: To use 8 CPUs, you may need around 24GB or RAM allocated to the VM!. So
adjust these numbers according to the RAM/CPU resources available on your host
machine.

- `SNAPCRAFT_BUILD_ENVIRONMENT_CPU=8 SNAPCRAFT_BUILD_ENVIRONMENT_MEMORY=24G snapcraft --debug`


### Gotcha #3: Could not get lock /var/lib/dpkg/lock-frontend
Sometimes, the `snapcraft --debug` command fails with a message like this:

```
E: Could not get lock /var/lib/dpkg/lock-frontend - open (11: Resource temporarily unavailable)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is another process using it?'
```

This happen from time to time. Apparently, Ubuntu's
`unattended-upgrades` is doing its thing. One option is to just drop
with a shell on the VM and simply wait for the upgrade process to
finish, or even, run apt-get upgrade yourself. It all else fails, it
may be worth to just delete the VM and recreate it.

- `multipass list` (to list the existing VMs)
- `multipass delete snapcraft-tizonia` (to mark for deletion the VM created by snapcraft to build tizonia)
- `multipass purge` (to remove the VMs that are marked for deletion)

### Tip #1: Install the snap locally

Once you get to the point where the build process is successful, it is the time
to start testing locally the new snap.

- `sudo snap install --dangerous tizonia_0.22.0_amd64.snap`

It is likely that the application does not work the first time, or that some
streaming services work but not others.

### Tip #2: Inspect the snap environment

You can for use these commands to inspect the snapped app environment
```
$ snap run --shell tizonia
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

juan@ubuntu1804:/home/juan/Documents$ env | grep HOME
HOME=/home/juan/snap/tizonia/297

juan@ubuntu1804:/home/juan/Documents$

```

### Tip #2: Iterate with `snapcraft try` and `sudo snap try prime`

With `snapcraft try` you get a local copy of the `prime` directory that can be
used to inspect the location and contents of files in the snap. This is
obviously useful when you are trying to find out why things don't quite work.

With `sudo snap try prime` you can install the snap using the contents of the
`prime` directory generated by `snapcraft try`. This allows you to change some
things quickly and give them a try, without rebuilding the entire snap.

### Tip #3: Use `snappy-debug.security scanlog` to troubleshoot AppArmor denials

First, you need to install `snappy-debug`

```
$ snap install snappy-debug

```

Then you run this on a terminal window

```
$ snappy-debug.security scanlog

```

then you run tizonia on another terminal window.

The output of `snappy-debug` will give you some clues as to what to do to
resolve some of the problems:

```
─ snappy-debug.security scanlog
INFO: Following '/var/log/syslog'. If have dropped messages, use:
INFO: $ sudo journalctl --output=short --follow --all | sudo snappy-debug
sysctl: permission denied on key 'kernel.printk_ratelimit'


= AppArmor =
Time: Jun  2 23:11:13
Log: apparmor="DENIED" operation="open" profile="snap.tizonia.tizonia" name="/proc/29913/mounts" pid=29913 comm="audio_source" requested_mask="r" denied_mask="r" fsuid=1000 ouid=1000
File: /proc/29913/mounts (read)
Suggestions:
* adjust program to not access '@{PROC}/@{pid}/mounts'
* add one of 'mount-observe, network-control' to 'plugs'

= AppArmor =
Time: Jun  2 23:12:51
Log: apparmor="DENIED" operation="open" profile="snap.tizonia.tizonia" name="/proc/30203/mounts" pid=30203 comm="audio_source" requested_mask="r" denied_mask="r" fsuid=1000 ouid=1000
File: /proc/30203/mounts (read)
Suggestions:
* adjust program to not access '@{PROC}/@{pid}/mounts'
* add one of 'mount-observe, network-control' to 'plugs'
```


# Snap TODO items

This is a list of things that currently either need some work and are just
definitely broken in the snap. Migration to Meson is the first task in the list
as it is the one that should be accomplished first in order to update the Snap
package to the latest version of Tizonia (0.22.0).

- [ ] Test and fix Chromecast functionality (optional for the next release).
- [ ] Test and fix MPRISv2 functionality (optional for the next release).
- [ ] Test and fix tab completion (optional for the next release).

# Useful links

- Tizonia's main repository:
  - https://github.com/tizonia/tizonia-openmax-il
- Building Tizonia with Meson:
  - https://github.com/tizonia/tizonia-openmax-il/blob/master/BUILDING_with_meson.md
- About Snaps:
  - https://snapcraft.io/docs/getting-started
- About the Snapcraft tool:
  - https://snapcraft.io/docs/snapcraft-overview
- Snap packaging for C/C++ applications:
  - https://snapcraft.io/docs/c-c-applications
- Snap packaging for Python applications:
  - https://snapcraft.io/docs/python-apps
- The Meson Snap plugin:
  - https://snapcraft.io/docs/meson-plugin
- Iterating over a Snap build
  - https://snapcraft.io/docs/iterating-over-a-build
- Debugging building Snaps
  - https://snapcraft.io/docs/debugging-building-snaps
- Snapcraft try
  - https://snapcraft.io/blog/development-tips-and-tricks-snap-try-and-snapcraft-pack
- Environment Variables that Snapcraft consumes:
  - https://forum.snapcraft.io/t/environment-variables-that-snapcraft-consumes/9416
- Gotchas and Tips:
  - https://daveparrish.net/posts/2019-08-16-Gotchas-When-Creating-Snap-Package-For-Haskell-Program.html
  - https://readyspace.co.id/en/faster-snap-development-additional-tips-and-tricks/
- Snap layouts
  - https://forum.snapcraft.io/t/snap-layouts/7207
- Snapcraft-preload
  - https://github.com/sergiusens/snapcraft-preload
  - https://forum.snapcraft.io/t/python-multiprocessing-sem-open-blocked-in-strict-mode/962/18
