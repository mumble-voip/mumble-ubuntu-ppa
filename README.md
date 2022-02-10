# Mumble Ubuntu PPA template package


```bash
./ppagen.bash 1.4.230 focal 1 --release --dry-run
#             ^<mumbleversion>
#                     ^ <ubuntu version>
#                           ^ <ppa release version> (appended to mumble version)
#                             ^ release or snapshot
#                                       ^ dry run to build without upload
```

1. Download release source tar from dl.mumble.info
2. Verify tar signature
3. Extract sources from tar
4. Clone mumble-ubuntu-ppa
5. Apply distro patches from it
6. Prepare changelog with `debchange` (or `dch` alias)
7. Build deb package with `debuild`
8. Upload with `dput`

Workaround for Windows WSL2 fakeroot not working:

```bash
sudo update-alternatives --set fakeroot /usr/bin/fakeroot-tcp
```

The source tar is signed with gpg, so gpg keys have to be imported.

```bash
gpg --import mumble-auto-build-2022.asc` from <https://github.com/mumble-voip/mumble-gpg-signatures>.
```

This git repository contains a template Debian package that we use to
generate the Mumble PPAs that live on Launchpad:

| PPA | Description |
| --- | --- |
| [release][ppa-release] | Current stable release of Mumble and Mumble Server |
| [snapshot][ppa-snapshot] | Current development snapshot of Mumble and Mumble Server for testind and bleeding-edge features and fixes |

The provided packages are the client `mumble` and the server `mumble-server`.

## Context

PPA stands for Personal Package Archive.

While a distribution like Ubuntu prepares and offers official packages (builds) maintained by a dedicated maintainer these packages can become outdated compared to the upstream source project because of the fixed release cycles of the distribution.

Some distributions offer backports package repositories that backport newer program versions to previous OS distribution versions. But a single backports repository will contain a lot of newer packages instead of just the ones you want to explicitly use newer versions of. Manually managing those (with a lower auto-install priority) is possible, but manual work.

PPAs allow third parties like upstream official projects to host their own package archives which can be added to an installations package sources. These PPAs are often more focused on single software packages.

For Mumble we provide a [release][ppa-release] and a [snapshot][ppa-snapshot] repository (under [~mumble][ppa]). The release PPA holds current stable releases for current and older Ubuntu releases. The snapshot PPA holds current development snapshots for testing and bleeding-edge features and fixes.

### PPA package building

Ubuntu uses the Debian apt deb package system.

This repository holds deb package information and scripts for building deb packages, and for uploading them to the PPA.

### Our infrastructure

In our internal build system we have a job named 'Mumble Ubuntu PPA Submitter' that prepares a package source tar ball that is uploaded to launchpad, built on the launchpad build system, and published in the PPAs for numerous Ubuntu versions.

## General Idea

The idea of this package is to be a base from which packages for various
Ubuntu PPA builds can be built from.

All the entries made by the Mumble team in the changelog are made for
a special release called 'template'. For example, the latest release
as of this writing is 1.2.4~rc1-8-gb115a29-0~template1 and the whole
changelog entry is:

    mumble (1.2.4~rc1-8-gb115a29-0~template1) template; urgency=low

      * Drop version requirement on libzeroc-ice-dev to build on precise.
      * Use bundled CELT codecs to build on raring.
      * Add CONFIG+=bundled-opus to force the use of the bundled Opus library.

     -- Mikkel Krautz <mikkel@krautz.dk>  Sat, 03 Feb 2013 14:39:40 +0100

All modifications made in the base template should target the template release
and use the debian version '0~template1' (and bump if appropriate).

Every time the Mumble team updates the template package, one of these template
entries will be added. The package version number that is used is simply a
reflection of the current Mumble package at the time of the update.

When building a source package for upload to Launchpad, the `ppagen.bash` script
(which also lives in this repository) can be used. For example, to generate
a package using the template, use:

    ./ppagen.bash 1.2.4~rc1-8-gb115a29 quantal 1 --dry-run  # drop --dry run to upload

and without dry run, uploaded to the mumble-snapshot PPA
(which lives at <https://launchpad.net/~mumble/+archive/snapshot>):

    ./ppagen.bash 1.2.4~rc1-8-gb115a29 quantal 1 --snapshot  # use --release to upload
                                                             # to the mumble-release PPA

The `ppagen.bash` script will, when running the above command:

* Download the Mumble tarball for the given version (and a GPG .sig)
* Verify the signature
* Clone the mumble-ubuntu-ppa repository into the 'debian' folder
  in the root of the source tree.
* Add a new changelog entry for the PPA build.
* Upload it to the mumble-snapshot PPA (<https://launchpad.net/~mumble/+archive/snapshot>)

The changelog entry that the script adds will be of the form:

    mumble (1.2.4~rc1-8-gb115a29-1~ppa1~quantal1) quantal; urgency=low

      * PPA Upload of 1.2.4~rc1-8-gb115a29 snapshot for Ubuntu quantal

     -- Mikkel Krautz <mikkel@krautz.dk>   Sun, 03 Feb 2013 15:19:02 +0100

with the latest template changelog entry exactly below it:

    mumble (1.2.4~rc1-8-gb115a29-0~template1) template; urgency=low

      * Drop version requirement on libzeroc-ice-dev to build on precise.
      * Use bundled CELT codecs to build on raring.
      * Add CONFIG+=bundled-opus to force the use of the bundled Opus library.

     -- Mikkel Krautz <mikkel@krautz.dk>  Sat, 03 Feb 2013 14:39:40 +0100

That is, the changelog entry that is added by the `ppagen.bash` script is merely
a temporary indicator of the fact that a PPA build was performed (oh, and it of
course also provides the version number to use for the package and the release
to target).

[ppa]: https://launchpad.net/~mumble
[ppa-release]: https://launchpad.net/~mumble/+archive/ubuntu/release
[ppa-snapshot]: https://launchpad.net/~mumble/+archive/ubuntu/snapshot
