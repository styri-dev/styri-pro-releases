# Styri Pro releases

Downloads for **Styri Pro**, for sites that can't pull from a registry or run air-gapped.
This repository holds no source code: only [releases](https://github.com/styri-dev/styri-pro-releases/releases)
and the key that signs them.

Without a license, Styri Pro is [Styri Core](https://github.com/styri-dev/styri): up to three robots.
A Styri Pilot license unlocks up to 20 robots on one site. Setup and licenses:
[the Pilot guide](https://github.com/styri-dev/styri/blob/main/docs/PILOT.md).

## What each release has

| File | For |
|---|---|
| `styri-pro-<tag>-linux-amd64.tar.gz`, `…-arm64.tar.gz` | Docker without registry access: the image, for `docker load` |
| `zarf-package-styri-pro-amd64-<tag>.tar.zst`, `…-arm64…` | Air-gapped Kubernetes: the image and its manifests, signed |
| `SHA256SUMS.txt` | Checksums for all four |
| `cosign.pub` | The public key the Zarf packages are signed with (also in this repository) |

Pick the architecture of the site's machine: `uname -m` says `x86_64` for amd64, `aarch64` for arm64.
With registry access you don't need any of this: `docker pull ghcr.io/styri-dev/styri-pro:<tag>`.

## Docker, without registry access

```bash
sha256sum -c --ignore-missing SHA256SUMS.txt
docker load < styri-pro-v2.0.1-linux-amd64.tar.gz
```

Then run it as the [Pilot guide](https://github.com/styri-dev/styri/blob/main/docs/PILOT.md) shows.

## Air-gapped Kubernetes, with Zarf

Carry in the Zarf package for the site's architecture, `SHA256SUMS.txt`, `cosign.pub`, and Zarf's own
init package for the same Zarf version (v0.88.0) and architecture. Run `zarf init` once on the cluster,
then:

```bash
sha256sum -c --ignore-missing SHA256SUMS.txt
zarf package deploy zarf-package-styri-pro-amd64-v2.0.1.tar.zst --key cosign.pub \
  --set-variables CONFIG_FILE=styri.yaml --set-variables ADMIN_PASSWORD=... --confirm
```

`--key cosign.pub` makes Zarf refuse a package that isn't signed with our key.

| Variable | Default |
|---|---|
| `CONFIG_FILE` | required: the site's `styri.yaml` (robots, control, VDA 5050) |
| `ADMIN_PASSWORD` | empty: the first-run screen sets one |
| `ROBOT_PASSWORDS` | `{}`: VDA 5050 broker passwords by `password_env` name, like `{AGV07_PASS: secret}` |
| `ROBOT_NAME` | `robot` |
| `ROS_DOMAIN_ID` | `0` |
| `BATTERY_TOPIC` / `DIAGNOSTICS_TOPIC` | `/battery_state` / `/diagnostics` |
| `MAX_LINEAR` / `MAX_ANGULAR` | `0.5` / `1.0` |

Styri runs in the `styri` namespace with host networking; open `http://<node-ip>:8080`.
