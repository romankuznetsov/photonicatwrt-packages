# photonicatWrt packages

Third-party opkg packages built against **photonicatWrt**'s own toolchain and
kernel, served as a feed from GitHub Pages.

## Why this exists

photonicatWrt is built on OpenWrt 25.12 but kept **opkg**, while upstream
OpenWrt moved to apk at that version. That combination strands it:

| Source | Problem |
| --- | --- |
| OpenWrt 24.10.8 | right format (`.ipk`), wrong kernel — its kmods are built against 6.6.144, the device runs 6.12.91 |
| OpenWrt 25.12.x | right kernel era, wrong format — `.apk` only, which opkg cannot read |
| ImmortalWrt 25.12 | same apk problem, and carries no amneziawg at all |
| the vendor feed | ~1400 packages against upstream's ~8700 |

A kernel module carries a `vermagic` string that must match the running
kernel exactly, so for kmods there is no substitute for building against
photonicatWrt's own tree.

## Using the feed

Add to `/etc/opkg/customfeeds.conf` on the device:

```
src/gz pcat_pkgs https://romankuznetsov.github.io/photonicatwrt-packages/26.04.1/aarch64_generic
src/gz pcat_kmod https://romankuznetsov.github.io/photonicatwrt-packages/26.04.1/rockchip-armv8/6.12.91
```

then `opkg update`.

**The second line contains a kernel version, and that is deliberate.** Kernel
modules only load on the kernel they were built against. When a vendor
firmware update changes it, that URL stops resolving and `opkg update` says
so — which is far better than installing a module that then refuses to load.
Check `uname -r` and point the line at the matching path.

Userspace packages are not kernel-bound, so the first line survives firmware
updates.

## How it is built

The expensive part is done once. `make target/sdk/install` produces an SDK
carrying photonicatWrt's prebuilt toolchain and kernel headers; packages are
then compiled against that in minutes rather than rebuilding a 2h30m tree
each time.

```
build-sdk.yml    ~2h30m, run only when the vendor firmware changes
                 -> openwrt-sdk-*.tar.zst as a release asset,
                    tagged with the kernel version and source commit

build-feed.yml   minutes
                 -> .ipk, indexed, deployed to GitHub Pages
```

`packages.list` is the whole interface: add a line, push, and the package
joins the feed.

## Packages are not signed

opkg only verifies signatures when `check_signature` is set, and photonicatWrt
does not set it. Signing would require a usign public key in
`/etc/opkg/keys` on every device — the same shape of problem that once made
packages unresolvable on another router here, because a key the config
management could not carry aborted the whole install transaction. GitHub Pages
is HTTPS-only, so the transport is authenticated regardless.
