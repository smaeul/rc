## Overview

This is the set of scripts and service definitions used on all of [my][smaeul]
Linux boxes. You can use this repository as a starting point for your machines,
or you can copy individual service directories as needed. Keep in mind that the
format extensions below are provided by the scripts in this repository, not by
s6-rc, so you will have to "de-sugar" run scripts taken from this repository.

These scripts and service directories were developed on gentoo-musl, but they
should work on any Linux distribution (possibly with minor modification).

For examples of user-level service definitions, also managed by similar scripts
and s6-rc, see [https://github.com/smaeul/rc-user][rc-user].

## Dependencies
1. The [s6][s6]/[s6-rc][s6-rc] software suite and dependencies.
2. Standard POSIX command-line utilities, plus `install`. These can be provided
   by e.g. coreutils/psmisc or busybox, though at the moment `kill` invocations
   are hardcoded to use busybox `kill` due to a bug in psmisc `kill`.

## Installation
1. Clone this repository to `/etc/rc`. Putting it somewhere else is fine, but
   requires modifying the `init` and `update` scripts to point to the correct
   location.
2. Configure the desired set of services (see below). A supervisor will be run
   for all services except those with a `disabled` marker, but only those
   services contained in the `default` bundle (and their dependencies) will be
   started at boot.
3. Create required users. The fd holder is run as `nobody`, and all
   automatically-generated logger services run under the `log` user (though
   this can be changed in the `update` script). Some services may require their
   own individual users; see their respective run scripts.
4. Run `/etc/rc/update` to compile the service database.
5. Add `init=/etc/rc/update` to your kernel command line (replacing any
   existing init path).
6. Reboot.

## Configuration

All of the [s6][s6] and [s6-rc][s6-rc] documentation applies here. In addition,
the `update` script implements a few extensions to the service directory
format.

### `disabled`

Presence of this file causes the service to be entirely excluded from the
compiled database. No supervisor will be created for it at runtime, but it will
not be available for dependency resolution.

### `instances`

This file contains a list of "instances" (variants) of the service, with one
instance name on each line. If this file is present, the `update` script
creates a copy of the service for each instance, suffixed with the (lightly
escaped) instance name, and with all occurrences of the string `%I` in the
service directory replaced by the instance name. The `update` script also
generates a bundle (named the same as the original service) containing all of
the instantiated services.

Note that if the `instances` file is present and empty, only an empty bundle is
created. This ensures dependencies are always satisfied, even if no instances
of the service are needed.

### `logdir`

The content of this file (one line) specifies a subdirectory of `/var/log`
where logs for this service are stored. If this file is present, the `update`
script creates a logger service and a pipeline connecting this service to its
logger.

## Troubleshooting

* An early getty is on tty4.
* Logs for everything that doesn't have a `logdir` file, including `s6-rc`
  itself, are in `/run/log/current`.

## Updates

Simply run `/etc/rc/update` after making changes to the source files.
Occasionally, you may wish to clean out old compiled databases. They are stored
in directories `/etc/rc/compiled.${TIMESTAMP}`. Be sure not to delete the
current (latest) compiled database, as it is required for s6-rc to function.

[rc-user]: https://github.com/smaeul/rc-user
[s6]: https://skarnet.org/software/s6/
[s6-rc]: https://skarnet.org/software/s6-rc/
[smaeul]: https://github.com/smaeul
