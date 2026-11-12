# credfeto-oxidized

[Oxidized](https://github.com/ytti/oxidized) on VM `oxidized.lan` (192.168.150.32): hourly configuration
backups of the network devices, committed to git and pushed to the private repo
`dnyw4l3n13/network-config`.

## Deploy

On the VM, as root:

```sh
git clone https://github.com/credfeto/credfeto-oxidized.git /root/credfeto-oxidized
cd /root/credfeto-oxidized
cp example.env .env
./update
```

The first `./update` creates two SSH key pairs in `DATA_DIR/keys` and prints the public keys:

- `devices_ed25519.pub`: add it to the read-only `backup` user on each device in `oxidized/router.db`.
- `deploy_ed25519.pub`: add it to `dnyw4l3n13/network-config` as a deploy key **with write access**.

Devices are listed in `oxidized/router.db`; settings are in `oxidized/config`. Edit them here and run
`./update` again.

## Build Status

| Branch  | Status                                                |
|---------|-------------------------------------------------------|
| main    | [![Build: Pre-Release][pre-release-img]][pre-release] |
| release | [![Build: Release][release-img]][release]             |

## Changelog

View [changelog][changelog].

## Contributing

See [contributing guidelines][contributing].

## Security

See [security policy][security].

## Licence

This project is licensed under the [MIT Licence][licence].

[changelog]: CHANGELOG.md
[contributing]: CONTRIBUTING.md
[licence]: LICENSE
[pre-release-img]: https://github.com/credfeto/cs-template/actions/workflows/build-and-publish-pre-release.yml/badge.svg
[pre-release]: https://github.com/credfeto/cs-template/actions/workflows/build-and-publish-pre-release.yml
[release-img]: https://github.com/credfeto/cs-template/actions/workflows/build-and-publish-release.yml/badge.svg
[release]: https://github.com/credfeto/cs-template/actions/workflows/build-and-publish-release.yml
[security]: SECURITY.md
