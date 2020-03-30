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

### Gotcha #1
Multipass VMs are stored in the 'vault' directory which
lives under `/var/snap/multipass/common`. If you have a /root partition with
limited space (like I do), Multipass can fill up the partition. Unfortunately,
at the time of writing, Multipass does not provide a configuration option to
customize the location of VM files in the system. See the following links for information on possible workarounds to this problem.:
- https://github.com/canonical/multipass/issues/1215
- https://forum.snapcraft.io/t/moved-var-lib-snapd-into-home-snapd-and-symlicked-back-snaps-fail-to-start/15272/2

### Gotcha #2
Tizonia requires quite a bit of RAM during the build process. By default,
Multipass creates VMs with 1GB or RAM. This won't cut it while building Tizonia
and the compilation will fail. Use the `SNAPCRAFT_BUILD_ENVIRONMENT_MEMORY`
environment variable to instantiate Multipass VMs with at least 8GB of RAM.

E.g.: from the top of the `tizonia-snap` repo
- `SNAPCRAFT_BUILD_ENVIRONMENT_MEMORY=24G snapcraft --debug`

(NOTE: If you have already issued an `snapcraft --debug` with the default RAM,
adding the `SNAPCRAFT_BUILD_ENVIRONMENT_MEMORY` environment variable afterwards
will not work as the VM memory cannot be modified at that point. You will need
to explicitly destroy the existing VM by calling `snapcraft clean`, and then
start over).

### Gotcha #3
Sometimes, the `snapcraft --debug` commands fails with a message like this:

```
E: Could not get lock /var/lib/dpkg/lock-frontend - open (11: Resource temporarily unavailable)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is another process using it?'
```
This may happen when the Multipass VM has not been used for a while and Ubuntu's `unattended-upgrades` is doing its thing.
It may be worth just deleting the VM and recreating it.

- `multipass list` (to list the existing VMs)
- `multipass delete snapcraft-tizonia` (to mark for deletion the VM created by snapcraft to build tizonia)
- `multipass purge` (to remove the VMs that are marked for deletion)

# Snap TODO items

This is a list of things that currently either need some work and are just
definitely broken in the snap. Migration to Meson is the first task in the list
as it is the one that should be accomplished first in order to update the Snap
package to the latest version of Tizonia (0.21.0).

- [ ] Migrate to Meson (this should be a must for the next release).
- [ ] Test and fix Chromecast functionality (optional for the next release).
- [ ] Test and fix MPRISv2 functionality (optional for the next release).
- [ ] Test and fix tab completion (optional for the next release).
