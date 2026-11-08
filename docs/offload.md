# Offload notes

The generic device leaves an offload seam (`iptv_offload_ops`) but does not
implement a backend. This file records the open questions and the archived
Airoha reference.

## Current state

`struct iptv_offload_ops` contains only:

```c
void (*probe)(struct net_device *dev, void *priv);
```

`iptv_offload_register()` stores the ops and private pointer on the device.
No generic TC/flowtable plumbing is present, and `HW_TC` is not advertised,
so the flowtable does not try to install flows on the device.

## Why software demux loses offload

The PON-native implementation translates the multicast VLAN to the service
VLAN in the PON driver, then lets PPE resolve the bridge egress from the
inline tag and install an L2B entry. The generic demux intercepts frames in
the receive handler and strips the tag in software, so the PPE classification
that would normally match the VLAN-over-PON path is bypassed.

The loss is limited to IPTV traffic; Internet flows do not pass through the
demux.

## Archived Airoha reference

`airoha/` contains the temporary HG5582A implementation:

- target patches 9994-9997 modify `airoha_eth` / `airoha_ppe`;
- `airoha-xpon` patches add the PON service netdev and use the exported
  helpers;
- `luci-app-onu` files provide the userspace side.

It is a reference, not a dependency of the generic module. It shows that in
the PON case the classification/forwarding work naturally belongs in the
lower driver, not in the demux netdev.

## Options

A. Keep the seam in `kmod-iptv`
   - The module hands flows to a backend via TC (`ndo_setup_tc`,
     `TC_SETUP_FT`) or a driver-specific callback.
   - The backend must understand the lower device's VLAN/PPE view.

B. Keep the seam in the lower driver
   - `kmod-iptv` only creates the netdev and configures VIDs.
   - The PON driver owns VLAN translation and PPE offload; the demux is a
     software fallback or a control-plane view.
   - Closer to the archived Airoha implementation, but couples the demux to
     the lower driver.

## Open questions

- Does the flowtable/TC path have enough information after the tag is
  stripped in software?
- Should the demux expose the VID map to the lower driver so it can install
  hardware rules?
- Is a single netdev still the right abstraction when the hardware wants to
  keep the two VLANs separate?
- What is the multicast story? PPE port-mask offload is not implemented in
  the archived variant either.
