[![Snap Status](https://build.snapcraft.io/badge/tizonia/tizonia-snap.svg)](https://build.snapcraft.io/user/tizonia/tizonia-snap)

# Tizonia Snap

A Tizonia 'snap' package is currently available in the 'candidate' channel. To
install, make sure you have
[snapd](https://docs.snapcraft.io/core/install?_ga=2.41936226.1106178805.1514500852-128158267.1514500852)
installed on your system. Once 'snapd' is available, use this command to
install Tizonia:

```bash

$ sudo snap install --candidate --devmode tizonia

```

> NOTE: '--devmode' disables snap confinement. A Tizonia snap that works with
> 'strict' confinement will be available very soon.

# Building

The Snapcraft build service is being used to produce automated builds of the
Tizonia snap for 'amd64', 'i386', and 'armhf'. However, to build the snap
package locally, use the 'cleanbuild' method.


```bash

# Install lxd and get it started
$ sudo apt install lxd lxd-client zfsutils-linux
$ sudo lxd init

# From the top of this repo, build Tizonia inside a lxd container.
$ sudo snapcraft cleanbuild --debug

```

> NOTE: the --debug flag is there to get a shell into the container when the
> build fails. Useful for troubleshooting.

# License

MIT
