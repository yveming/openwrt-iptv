# Archived Airoha PON IPTV implementation

This directory preserves the temporary IPTV implementation used for Airoha
PON devices (HG5582A and similar). It is not the generic `kmod-iptv` path and
is not built by this repository.

## Why it is archived

- It modifies `airoha_eth`, `airoha_ppe` and `airoha-xpon` in ways that break
  the PON driver layering.
- It depends on out-of-tree target patches (9994-9997) and on local commits in
  `openwrt-pon-drivers`.
- It was kept as a reference for the future offload work, not as a supported
  package.

Do not apply these patches to the main tree and do not package this directory.
The maintained generic implementation lives in `kernel/iptv` and
`luci/luci-app-iptv`.

## Contents

| Path | Target | Notes |
| --- | --- | --- |
| `target/` | `target/linux/airoha/patches-6.18/` | Kernel-side VLAN/PPE plumbing |
| `xpon/` | `feeds/pon_drivers/airoha-xpon/` | PON service netdev |
| `luci-app-onu/` | `feeds/onu_config/luci-app-onu/` | Extracted userspace files |

## Dependency

```
target 9995/9997  --exports/hooks-->  airoha-xpon
   (airoha_eth/airoha_ppe)              (service netdev)
        ^                                    ^
        |                                    |
        +----------- luci-app-onu -----------+
                     (configuration)
```

The target patches provide the exported helpers and the `rx_service` hook; the
xpon module consumes them and creates the logical service netdev. They are not
duplicates of each other.

See `MANIFEST.md` for base commits and known limitations.
