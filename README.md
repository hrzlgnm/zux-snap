# zux snap

Packages [zux](https://github.com/hrzlgnm/zux) as a Snap. This repository
only contains packaging metadata and CI; no zux code lives here.

The snap ships the prebuilt `zux_linux_x64` binary from the upstream GitHub
release (the same artifact the AUR `-bin` package uses). Its sha256 is verified
against the inline digest GitHub computes for every release asset. The release
tag is pinned via `source-tag` in `snap/snapcraft.yaml`, which CI rewrites with
the [pin-upstream-tag](.github/actions/pin-upstream-tag) action before building;
the version is derived from the checked-out tag.

## Install

Once published:

```console
sudo snap install zux
```

## Releasing

1. Tag a release on `hrzlgnm/zux` (e.g. `v1.8.0`) and let its
   release workflow finish; the upstream **Update Snap** workflow dispatches
   this repository's release automatically.
2. To repackage manually, run this repository's **Release snap** workflow with
   `tagName` set to that tag (it defaults to the latest upstream release when
   unset, as CI does).
3. CI builds the snap in LXD, uploads it as an artifact, smoke tests it in an
   Ubuntu 24.04 LXC container (`zux --help` must print usage), and then uploads
   it to the `stable` channel of the Snap Store via `snapcraft upload`.

## Store setup (one-time)

- Register the name: `snapcraft register zux`
- Create exportable store credentials and add them as the
  `SNAPCRAFT_STORE_CREDENTIALS` repository secret:

  ```console
  snapcraft export-login --snaps=zux --channels=stable login-file
  gh secret set SNAPCRAFT_STORE_CREDENTIALS < login-file
  rm login-file
  ```

  Note: snapcraft 9 removed support for exporting to stdout (`-`); always
  pass a file name.

If publish credentials are absent, CI still builds and smoke tests the snap;
the publish step is skipped.

## Notes

- Confinement is `strict`; mDNS discovery works through the `network`
  interface plug.
- In-snap auto-updates are disabled by design (upstream self-disables the
  updater for non-bundled binaries); updates are delivered via snap refresh.
