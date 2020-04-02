Tizonia's Snap Development
==========================

# Overview

According to the Snapcraft statistics of the Tizonia Snap, this package is
actively used by around 2500 people. So lots of people would be glad to see the
package updated and in sync with the latest fixes in the main repo
(https://github.com/tizonia/tizonia-openmax-il).

The Tizonia Snap package is lagging a few versions behind the latest release of
Tizonia (0.18.0 vs 0.21.0). So it is in need of a refresh. The following is a
list of resources and information on the status of the Tizonia Snap
package. This might guide people wanting to help with the Snap packaging
activities of the Tizonia project.

# Resources

Tizonia is a hybrid application that confluates code written in C, C++ and
Python. While packaging Tizonia as a Snap (or in any other format for that
matter) it is important to keep this in mind.

Another important detail to know about is that Tizonia has recently added a
Meson build system in the main repository. However, the Snap package still uses
the old Autotools build system as of version 0.18.0. Note that the Meson build
system co-exists with Autotools in the main repo, but Autotools will be
deprecated at some point in the near future. So for the next refresh of the
Snap package, it is recommended to migrate it to Meson.

- Tizonia's main repository: https://github.com/tizonia/tizonia-openmax-il
- Building Tizonia with Meson: https://github.com/tizonia/tizonia-openmax-il/blob/master/BUILDING_with_meson.md
- Learning about Snaps: https://snapcraft.io/docs/getting-started
- Learning about the Snapcraft tool: https://snapcraft.io/docs/snapcraft-overview
- Snap packaging for C/C++ applications: https://snapcraft.io/docs/c-c-applications
- Snap packaging for Python applications: https://snapcraft.io/docs/python-apps
- The Meson Snap plugin: https://snapcraft.io/docs/meson-plugin
- About Snap packaging with Multipass:
  - https://snapcraft.io/docs/iterating-over-a-build
  - https://readyspace.co.id/en/faster-snap-development-additional-tips-and-tricks/
- Some gotchas: https://daveparrish.net/posts/2019-08-16-Gotchas-When-Creating-Snap-Package-For-Haskell-Program.html

# Workflow (Ubuntu 18.04 assumed)

## Pre-requisites
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

### Tip #1
Multipass VMs are stored in the 'vault' directory which
lives under `/var/snap/multipass/common`. If you have a /root partition with
limited space (like I do), Multipass can fill up the partition. Unfortunately,
at the time of writing, Multipass does not provide a configuration option to
customize the location of VM files in the system. See the following links for information on possible workarounds to this problem.:
- https://github.com/canonical/multipass/issues/1215
- https://forum.snapcraft.io/t/moved-var-lib-snapd-into-home-snapd-and-symlicked-back-snaps-fail-to-start/15272/2

### Tip #2
Tizonia requires quite a bit of RAM during the build process. By default,
Multipass creates VMs with 1GB or RAM. This won't cut it while building Tizonia
and the compilation will fail. Use the `SNAPCRAFT_BUILD_ENVIRONMENT_MEMORY`
environment variable to instantiate Multipass VMs with at least 8GB of RAM.

E.g.: from the top of the `tizonia-snap` repo
- `SNAPCRAFT_BUILD_ENVIRONMENT_MEMORY=16G snapcraft --debug`

(NOTE: If you have already issued an `snapcraft --debug` with the default RAM,
adding the `SNAPCRAFT_BUILD_ENVIRONMENT_MEMORY` environment variable afterwards
will not work as the VM memory cannot be modified at that point. You will need
to explicitly destroy the existing VM by calling `snapcraft clean`, and then
start over).

While at it, if you have enough CPUs on your machine, you can also increase the
number of CPUs that Multipass will allocate on the VM (the default is 2
CPUs). To make use of the additional CPUs, you need to modify the the number of parallel jobs
that ninja will use during the build. This value is 1 in the snapcraft.yaml, to avoid running out of memory.

E.g.:
```
ninja -j1 -C build --> ninja -j8 -C build

```

But REMEMBER: The more CPUs used in parallel by ninja, the more RAM required
for the overall build process.


E.g.: To use 8 CPUs, you may need a whopping 24GB or RAM allocated to the
VM!. So adjust these numbers according to the RAM/CPU resources available on
your host machine.

- `SNAPCRAFT_BUILD_ENVIRONMENT_CPU=8 SNAPCRAFT_BUILD_ENVIRONMENT_MEMORY=24G snapcraft --debug`


### Tip #3
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

### Tip #4

Once you get to the point where the build process is successful, it is
the time to start testing locally the new snap.

- `sudo snap install tizonia_0.21.0_amd64.snap`

It is likely that the applicaiton does not work the first time, or
that some streaming services work but not others.

You can for use these commands to inspect the snapped app environment
```
$ snap run --shell tizonia
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

juan@ubuntu1804:/home/juan/Documents$ env | grep HOME
HOME=/home/juan/snap/tizonia/297

juan@ubuntu1804:/home/juan/Documents$ 

```

More information on debugging the build process and on iterating over a build:
- https://snapcraft.io/docs/debugging-building-snaps
- https://snapcraft.io/docs/iterating-over-a-build

# Snap TODO items

This is a list of things that currently either need some work and are just
definitely broken in the snap. Migration to Meson is the first task in the list
as it is the one that should be accomplished first in order to update the Snap
package to the latest version of Tizonia (0.21.0).

- [ ] Migrate to Meson (this should be a must for the next release).
- [ ] Test and fix Chromecast functionality (optional for the next release).
- [ ] Test and fix MPRISv2 functionality (optional for the next release).
- [ ] Test and fix tab completion (optional for the next release).
