[![Snap Status](https://build.snapcraft.io/badge/tizonia/tizonia-snap.svg)](https://build.snapcraft.io/user/tizonia/tizonia-snap)

# Tizonia Snap

A 'snap' package is now available for download from the snap store ('stable'
channel). To install, make sure you have
[snapd](https://docs.snapcraft.io/core/install?_ga=2.41936226.1106178805.1514500852-128158267.1514500852)
installed on your system. Once 'snapd' is available, use this command to
install Tizonia:

```bash

$ sudo snap install tizonia

```

## Configuration

To use *Spotify*, *Google Play Music*, *SoundCloud*, and *Dirble*, introduce
your credentials in Tizonia's config file (see instructions inside the file for
more information):

```bash

    $ mkdir -p $HOME/snap/tizonia/current/.config/tizonia
    $ cp /var/lib/snapd/snap/tizonia/current/etc/xdg/tizonia/tizonia.conf $HOME/snap/tizonia/current/.config/tizonia

    ( now edit $HOME/snap/tizonia/current/.config/tizonia )

```

# Building

The Snapcraft build service is being used to produce automated builds of the
Tizonia snap for 'amd64', 'i386', and 'armhf'. However, to build the snap
package locally, the 'cleanbuild' method can be used.

```bash

# Install lxd and get it started
$ sudo apt install lxd lxd-client zfsutils-linux && sudo lxd init

# This is in case your user is not in the lxd group already
$ sudo usermod -a -G lxd $USER && newgrp lxd

# From the top of this repo, build Tizonia inside a lxd container.
$ snapcraft cleanbuild --debug

```

> NOTE: the --debug flag is there to drop into a shell inside the container if the
> build fails. Useful for troubleshooting.

# License

[MIT](LICENSE)

# Tizonia's main repo

See [tizonia-openmax-il](https://github.com/tizonia/tizonia-openmax-il).
