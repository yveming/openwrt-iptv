# Manifest

Archived: 2026-11-08

## Base

- Source tree: `~/HG5582A/work/openwrt-pon-hg5582a`
- Branch: `hg5582a-work`
- HEAD: `ead24ef897` (2026-11-02)
- Driver feed: `feeds/pon_drivers`, origin `yveming/openwrt-pon-drivers`
- Driver base: `04adfaf`
- Local driver commits (5 ahead of origin/main):

```
a877ab6 airoha-xpon: expose PON service VLAN translation
8397135 airoha-xpon: add logical PON service interface
783e645 airoha-xpon: expose PCS auto-mode diagnostics
7947cf9 airoha-xpon: allow MAC changes on the PON service netdev
65d792b airoha-xpon: align service netdev with standard VLAN upper
```

- Target patches: `target/linux/airoha/patches-6.18/9994` through `9997`
- Userspace: `feeds/onu_config/luci-app-onu`

## Status

Temporary. Multicast PPE port-mask offload is not implemented. Real-hardware
validation of STB blind-plug, VLAN 3990 translation and PPE L2B entries was
still pending. Not for main-tree merge.

## Notes

- `783e645` (PCS diagnostics) is unrelated to IPTV; it is included because it
  is part of the same local branch.
- The xpon patches are `git format-patch` output against the driver base above.
- The target patches are snapshots from the HG5582A tree; patch numbers are
  meaningful only in that tree.
