# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

<!--
Please ADD ALL Changes to the UNRELEASED SECTION and not a specific release
-->

## [Unreleased]
### Security
- Host egress lockdown (`firewall/configure`, run by `update`): firewalld policies limit what the host and the containers may connect to (Oxidized: SSH to the devices in `oxidized/router.db` and to GitHub), with CDN-fronted destinations kept in ipsets refreshed every 10 minutes. `oxidized-firewall-sync.timer` re-applies it when the firewall config changes, and the daily `Repo: Update egress ranges` workflow refreshes the GitHub and Cloudflare ranges. Observation mode (log unmatched traffic, drop nothing) until `OXIDIZED_FIREWALL_ENFORCE=true`
### Added
- Oxidized stack (docker compose + watchtower) backing up the MikroTik switches hourly to dnyw4l3n13/network-config; ./update installs the config into DATA_DIR and creates the device and deploy SSH keys
### Fixed
- `./update` now re-runs itself after `git pull`, so changes to the update script take effect on the same run instead of the next one
- update: create the local backup repo with `main` as HEAD so the push goes to main instead of creating a master branch
- update: key comments use `uname -n` (Arch has no `hostname` binary)
- GitHub push failing with "invalid or unknown remote ssh hostkey": ship GitHub's SSH host keys as oxidized/known_hosts, installed into DATA_DIR/ssh and mounted read-only at /home/oxidized/.ssh
### Changed
- The host egress firewall is now one nftables table (`inet oxd_egress`) loaded atomically by `firewall/egress`, instead of firewalld policies and ipsets: the 10-minute refresh reloaded firewalld, and a firewalld killed mid-reload (by the OOM killer) left its block-everything table in place and cut the host off. `firewall/configure` removes the old policies and ipsets with one last reload
### Deprecated
### Removed
### Deployment Changes

<!--
Releases that have at least been deployed to staging, BUT NOT necessarily released to live.  Changes should be moved from [Unreleased] into here as they are merged into the appropriate release branch
-->

## [0.0.0] - Project created