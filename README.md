# zux snap

Builds [zux](https://github.com/hrzlgnm/zux) as a Snap. The
snap source is pulled directly from the upstream repository at the tag being
packaged; this repository only contains packaging metadata and CI.

## Install

Once published:

```console
sudo snap install zux
```

Local build (requires LXD):

```console
sg lxd -c "snapcraft --use-lxd"
sudo snap install ./zux_*.snap --dangerous
```

## Releasing

1. Tag a release on `hrzlgnm/zux` (e.g. `v1.8.0`) and let its
   release workflow finish; the upstream **Update Snap** workflow dispatches
   this repository's release automatically.
2. To repackage manually, run this repository's **Release snap** workflow with
   `tagName` set to that tag.
3. CI builds the snap in LXD, smoke tests it in an Ubuntu 24.04 container, and
   uploads it to the `stable` channel of the Snap Store.

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
