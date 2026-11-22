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

## Host firewall

`update` runs `firewall/configure`, which locks the host's egress down with firewalld policies, the same way as the DNS servers:

| Policy              | Traffic                     | Allowed                                                                                                                                                                       |
|---------------------|-----------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `oxd-egress`        | host -> anywhere            | DNS to the fleet's DNS servers, NTP to the router, HTTPS to the proxies, journald uploads to monitoring, GitHub, ghcr.io, Docker Hub, Cloudflare, chaotic-aur, pkgstats, the Arch WKD, ICMP |
| `oxd-egress-docker` | containers -> outside world | DNS, HTTPS to the proxies, SSH to the devices in `oxidized/router.db`, GitHub over SSH (the backup push), Docker Hub and Cloudflare (Watchtower)                               |

Ingress needs nothing extra: no ports are published, and SSH is left to the host's base configuration. A device added to `oxidized/router.db` is allowed by the next `./update`.

Two timers keep it current:

* `oxidized-egress-refresh.timer` (every 10 minutes) refreshes the firewalld ipsets for destinations without a stable address: names resolved from DNS, plus GitHub's and Cloudflare's published ranges (`firewall/github-egress-ranges` and `firewall/cloudflare-egress-ranges`).
* `oxidized-firewall-sync.timer` (every 15 minutes) pulls the checkout and re-runs `firewall/configure` when anything under `firewall/` changed since the commit it last applied. Container and device list changes still wait for `update`.

The `Repo: Update egress ranges` workflow regenerates the two range files daily and commits any change to main, refusing a list that has shrunk by more than a fifth.

`OXIDIZED_FIREWALL_ENFORCE` in `.env` is the only switch. Unset or anything other than `true` is observation mode: every rule is in place, but traffic that matches none of them still flows and is logged with the prefix `oxd-egress-unmatched:` or `oxd-egress-docker-unmatched:` (rate-limited to 30 a minute). Run in observation mode first and check the journal for anything legitimate that is not covered:

```sh
sudo journalctl -k --since -1d | grep -E 'oxd-egress(-docker)?-unmatched'
```

then set `OXIDIZED_FIREWALL_ENFORCE=true` and run `./update`.

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
