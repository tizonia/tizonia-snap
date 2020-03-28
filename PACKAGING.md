Tizonia's Snap release TODO list
================================

# Overview

According to the Snapcraft statistics of the Tizonia Snap, this package is
actively used by around 2500 people. So lots of people would be glad to see the
package updated and in sync with the latest fixes in the main repo
(https://github.com/tizonia/tizonia-openmax-il).

The Tizonia Snap package is lagging a couple of versions behind the latest
release of Tizonia (0.18.0 vs 0.20.0). So it is in need of a refresh. The
following is a list of resources and information on the status of the Tizonia
Snap package. This might guide people wanting to help with the Snap packaging
activities of the Tizonia project.

# Resources

Tizonia is a hybrid application that confluates code written in C, C++ and
Python. While packaging Tizonia as a Snap (are in any other format for that
matter) it is important to keep this in mind.

Another important detail to know about is that Tizonia has recently added a
Meson build system in the main repository. However, the Snap package still uses
the old Autotools build system as of version 0.18.0. The Meson build system is
still co-existing with Autotools in the main repo (will be deprecated at some
point). But for future refreshes of the Snap package, the migration to Meson is
the recommended option.

- Tizonia's main repository: https://github.com/tizonia/tizonia-openmax-il
- Building Tizonia with Meson: https://github.com/tizonia/tizonia-openmax-il/blob/master/BUILDING_with_meson.md
- Learning about Snaps: https://snapcraft.io/docs/getting-started
- Snap packaging for C/C++ applications: https://snapcraft.io/docs/c-c-applications
- Snap packaging for Python applications: https://snapcraft.io/docs/python-apps
- https://snapcraft.io/docs/meson-plugin
https://snapcraft.io/docs/iterating-over-a-build

# Snap TODO items

This is a list of things that currently either need some work and are just
definitely broken in the snap. Migration to Meson is the first task in the list
as it is the one that should be accomplished first in order to update the Snap
package to the latest version of Tizonia (0.20.0).

- [ ] Migrate to Meson (this should be a must for the next release).
- [ ] Test and fix Chromecast functionality (optional for the next release).
- [ ] Test and fix MPRISv2 functionality (optional for the next release).
- [ ] Test and fix tab completion (optional for the next release).
