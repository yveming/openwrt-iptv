# openwrt-iptv

Linux/OpenWrt IPTV netdevice: expose one untagged interface over a lower
device while keeping the operator's service and multicast VLANs inside the
driver.

This is a source repository. `kernel/` and the other package directories
follow OpenWrt package layout, so the same tree can also be used as an
OpenWrt feed.

## Layout

| Path | Package | Description |
| --- | --- | --- |
| `kernel/iptv/` | `kmod-iptv` | Kernel module, UCI config, init script and apply logic |
| `luci/luci-app-iptv/` | `luci-app-iptv` | Optional LuCI configuration page |
| `airoha/` | — | Archived Airoha PON implementation (reference only) |
| `docs/` | — | Design and offload notes |

## Status

- `kernel/iptv` and `luci/luci-app-iptv`: maintained generic
  implementation.
- `airoha/`: archived temporary HG5582A implementation. It modifies
  `airoha_eth` / `airoha_ppe` / `airoha-xpon` in ways that break the PON
  driver layering. Do not apply it to the main tree and do not package it.

## How it works

```sh
ip link add name itv link pon0 type iptv
echo 43 3990 > /sys/class/net/itv/vids
ip link set itv up
```

- TX: unicast uses the service VID; IPv4/IPv6 multicast uses the multicast VID.
- RX: frames carrying either VID are stripped and delivered untagged.
- Offload: `iptv_offload_ops` is a placeholder seam; no backend is implemented yet.

## Configuration

VLAN IDs and the uplink device live in `/etc/config/iptv`, shipped by
`kmod-iptv`:

```uci
config iptv 'config'
	option enabled '1'
	option uplink 'pon0'
	option name 'itv'
	option service_vid '43'
	option mcast_vid '3990'
	option macaddr ''
```

`kmod-iptv` creates `itv` at boot (`START=19`, before network). Select it
in the image config (`CONFIG_PACKAGE_kmod-iptv=y`); `luci-app-iptv` also
pulls it in through its package dependency. Address and bridging are
configured with the standard Network pages:

- routed: create an interface with device `itv` and protocol DHCP or static;
- bridged: add `itv` to a bridge device's port list.

No custom netifd protocol is required.

## Build

As an OpenWrt feed (optional):

```sh
echo 'src-git iptv https://github.com/yveming/openwrt-iptv.git' >> feeds.conf
./scripts/feeds update iptv
./scripts/feeds install kmod-iptv luci-app-iptv
```

Standalone kernel module:

```sh
make -C /lib/modules/$(uname -r)/build M=$PWD/kernel/iptv/src modules
```

## Caveats

- `kmod-iptv` creates `itv` before netifd starts. If `pon0` does not exist
  yet, add a retry or a hotplug rule that waits for `pon0`.
- `enabled=0` removes `itv`; any Network interface or bridge referring to it
  will fail until `kmod-iptv` is enabled again.
- The package name `luci-app-iptv` conflicts with the older bridge-based
  package in `pon_userspace`. Retire or rename one before installing both.
- `airoha/` is an archive, not a feed package, and is not built.

## License

- `kernel/iptv/src`: GPL-2.0-only (see `LICENSE`)
- `kernel/iptv/files` and `luci/luci-app-iptv`: Apache-2.0 (see
  `kernel/iptv/LICENSE.scripts` and `luci/luci-app-iptv/LICENSE`)
- `airoha/`: follows the licenses of the files it patches.
