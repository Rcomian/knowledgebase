[Home](../index.md)

## Catalyst

![Build by SBTS from the Noun Project](res/catalyst-splash.png)

### Aims

* Work as a self-contained build directory - all config and artifacts in one structured place
* Build main workstation off the workstation itself
* Build VM's for testing and specific services
* Build VM for catalyst building
* Build AWS and Azure instances
* Enable building for multiple machines
* Allow management of individual features
* Allow configuration of machine fleets

### Basics

Artifacts:

* Snapshot - the base gentoo portage software tree to use
  * Can download from gentoo, use the `portage`
* Stage 3 - the base stage that everything else comes from
  * Used to go around from stage 1 if necessary
  * Used to go straight to a stage 4 if possible
* Stage 4 - an actual useable rootfs, possible with kernel & initramfs
* catalyst.conf - overall catalyst configs and locations
* catalystrc - global variable exports
* specfile - the configuration of a build stage

### Snapshot management

One of the main goals of the project is to be able to rebuild a machine image using a new snapshot while avoiding any issues and cruft building up in the machine image.

To do this, I'd expect to always use an official gentoo snapshot rather than make our own, our own may have their own cruft in them and cause issues.

The bouncer for snapshots is: https://bouncer.gentoo.org/fetch/root/all/snapshots/portage-latest.tar.xz

#### Which type of snapshot (portage, not gentoo)

We should use `portage-` snapshots rather than `gentoo-` snapshots. This is because when you extract `gentoo-` snapshots, the directory inside has a date associated with it and the catalyst system doesn't find the directory correctly. This can be fudged by extracting the file, renaming the directory and re-compressing, but that's ugly.

An alternative is to use the `portage-` snapshot. This however still can't be found by catalyst by default, but at least it has a consistent name directory each time (`portage`).

To use the `portage-` snapshot we need to change our `catalyst.conf` file to set the following:

```bash
# the name of the directory within the chroot image
portdir="/var/db/repos/portage"
# The prefix of the repo in the snapshots directory
repo_name="portage"
```

#### Snapshot versioning

Snapshots are identified by their version number. They're located in the `artifacts/snapshots` directory and are found by combining the name in `catalyst.conf` with the requested version number and adding `.tar.xz` on the end (actually various extensions are tried and allowed). This means a specfile can specify a specific portage version, which seems good. However it also means that if the portage version changes, all the specfiles must also change. This definitely seems unnecesary. However, we *do* want to be able to revert back to a previous snapshot if a particular snapshot appears bad. This is absolutely essential as bad configurations do make their way into portage. We might want to freeze an image at an old snapshot for a while and work with it. This is not ideal but it *would* provide stability, if necessary, and we can evaluate and control the security implications, especially with gentoo's security scanning tools.

This means we need the ability to have multiple snapshots, but also easily be able to move spec files up to later snapshots.

An additional issue is availability. Snapshots are served by mirrors and not all mirrors are as up to date as each other. So if we download a latest portage, it's no gaurantee that we're not going backwards in time.

For this reason, we always need to download a dated snapshot explicitly (most recent = today-1d, possibly today-2d).

Once downloaded, a dated snapshot should be symlinked into whatever roles want that snapshot. The specfiles then reference those snapshot roles, not specific snapshots.

### Working within a single directory

It's essential that this project can at least in theory be brought under source control. To do that, all specfiles must be committed to git, but also all overlays, kernel configs, and the catalyst.conf and catalystrc files. The build outputs should be in the same directory, just like any well behaved build. Addition downloaded artifacts like stage files and snapshots should be easily placed, and identified but definitely not added to source control.

Because the build directory could be cloned or moved at will (especially to make use of additional storage), the various configurations that specify the absolute path need to be updated. This is mainly in the catalyst.conf file.

This process would itself be the a build step within the build process, necessary before catalyst is itself used in any way.

To do this, we set up an `init` script which creates the config files in the right location from a template and sets them up from the current directory. Other scripts reference the config from this location, so they won't work unless `init` has been called.


