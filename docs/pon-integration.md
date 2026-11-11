# PON/OpenWrt integration notes

This page records the packaging and bring-up pitfalls found while integrating
`kmod-iptv` into the Airoha PON image.  The data path and netdevice model are
described in `design.md`; hardware offload is discussed in `offload.md`.

## Required package set

For the current Airoha AN7581 image the IPTV runtime is:

```text
kmod-iptv
luci-app-iptv
luci-i18n-iptv-zh-cn
omcproxy
luci-app-omcproxy
luci-i18n-omcproxy-zh-cn
kmod-nf-nathelper-fcc
kmod-nf-nathelper-rtsp
```

Relationships:

- `luci-app-iptv` depends on `kmod-iptv`; the kernel package also owns
  `/etc/config/iptv`, the boot script and the apply logic.
- `luci-app-omcproxy` depends on `omcproxy`; the LuCI package alone does not
  configure multicast forwarding.
- `kmod-nf-nathelper-fcc` is a local FCC/NAT helper.
- `kmod-nf-nathelper-rtsp` comes from `xtables-addons` and is optional.

## `itv` configuration

`kmod-iptv` creates one untagged virtual interface on top of the PON uplink.
The VLANs stay inside the driver:

```uci
config iptv 'config'
	option enabled '1'
	option uplink 'pon0'
	option name 'itv'
	option service_vid '43'
	option mcast_vid '3990'
	option macaddr ''
```

Addressing and bridging are intentionally left to the standard Network pages:

- routed: create an interface with device `itv`, protocol DHCP or static;
- bridged: add `itv` to a bridge device's port list.

No custom netifd protocol is required.

## Multicast

`kmod-iptv` only performs VLAN demultiplexing.  Multicast forwarding is handled
by the bridge / IGMP proxy / multicast routing stack.  When `omcproxy` is used,
configure its upstream and downstream interfaces explicitly; installing
`luci-app-omcproxy` is not enough.

## NAT helper and the xtables trap

`kmod-nf-nathelper-rtsp` is built by `xtables-addons`.  On this PON kernel the
legacy xtables stack is disabled:

```text
# CONFIG_NETFILTER_XTABLES is not set
```

Building the RTSP helper therefore fails with unresolved `xt_register_target`
and similar symbols unless the build can see the xtables symbols.  The working
configuration is:

```text
CONFIG_PACKAGE_kmod-ipt-core=m
CONFIG_PACKAGE_kmod-nf-nathelper-rtsp=y
```

`kmod-ipt-core=m` is a **build-time** dependency only.  It is not installed in
the final rootfs, and the image must not contain:

```text
kmod-ipt-core
iptables
```

Do not set the whole `kmod-ipt-*` family to `=y`; that pulls the legacy
iptables stack into the image and is not needed by `itv`.

## Device profile and seed

Add the package set to the device profile's `DEVICE_PACKAGES` **and** select
the build symbols in the seed.  A profile entry alone is not enough if the
package is configured as `=m` and not otherwise selected.

The HG5582A profile is:

```text
Device/fiberhome_hg5582a
```

It must include the IPTV packages above; `luci-app-iptv` pulls `kmod-iptv`.

## First-image package omissions

The first PON HG5582A image was about 2 MiB smaller than the reference voice
image.  The missing packages were:

```text
asterisk-app-playtones
asterisk-app-record
asterisk-bridge-simple
asterisk-codec-alaw
asterisk-codec-g722
asterisk-codec-ulaw
asterisk-format-wav
asterisk-res-musiconhold
asterisk-res-rtp-asterisk
asterisk-sounds
luci-app-airoha-npu
kmod-nf-nathelper-rtsp
```

Only the last item is IPTV-specific; the others were voice/Asterisk runtime
packages.  The general lesson applies here too: compare the **per-device APK
database**, not just the seed or the target-level manifest.

## Verification

Build-time:

```sh
T=build_dir/target-aarch64_cortex-a53_musl/linux-airoha_an7581/target-dir-*
grep -E '^P:(kmod-iptv|luci-app-iptv|omcproxy|luci-app-omcproxy|kmod-nf-nathelper)' \
  "$T/lib/apk/db/installed"
```

Runtime:

```sh
for p in kmod-iptv luci-app-iptv luci-i18n-iptv-zh-cn \
  omcproxy luci-app-omcproxy luci-i18n-omcproxy-zh-cn \
  kmod-nf-nathelper-fcc kmod-nf-nathelper-rtsp
do
  apk info -e "$p" >/dev/null 2>&1 || echo "MISS $p"
done

apk info -e kmod-ipt-core >/dev/null 2>&1 && echo "UNEXPECTED kmod-ipt-core"
apk info -e iptables >/dev/null 2>&1 && echo "UNEXPECTED iptables"
```

## Known limitations

- Software-only forwarding; the offload seam has no backend.
- `itv` is created before netifd; if `pon0` is not up yet, add a retry or
  hotplug wait.
- `enabled=0` removes `itv`; interfaces or bridges referring to it fail until
  it is enabled again.
- The package name `luci-app-iptv` conflicts with the older bridge-based
  package; do not install both.
