# Design

`kmod-iptv` creates one virtual Ethernet device (`itv` by default) on top of a
lower device (normally `pon0`). The device is untagged; the two operator VLANs
stay inside the driver.

## Data path

- TX: unicast frames are tagged with the service VID; IPv4/IPv6 multicast
  frames are tagged with the multicast VID.
- RX: the lower device's receive path is intercepted with `rx_handler`.
  Frames carrying either VID are stripped and delivered untagged through
  `itv`; other frames continue through the normal path.
- `gro_cells` is used on the receive path.
- The lower device's carrier state is followed through a notifier; the device
  is removed when the lower device is unregistered.

## Configuration

The VIDs are per-device sysfs configuration:

```sh
echo 43 3990 > /sys/class/net/itv/vids
```

`0 0` disarms the demux and unregisters the receive handler. An unconfigured
device stays inert: RX passes frames through and TX drops them.

`kmod-iptv` owns `/etc/config/iptv`, creates the device at boot and writes
the sysfs file. Addressing and bridging are left to the standard Network
pages; no custom netifd protocol is required.

## Netdevice model

The device is a `rtnl_link_ops` link type:

```sh
ip link add name itv link pon0 type iptv
```

The current implementation supports a single instance. The receive handler
has no private-data argument, so the instance is published through an
RCU-protected pointer.

## Offload seam

`iptv_offload_ops` is exported through `iptv_offload_register()`. It currently
contains only a `probe` callback and is a placeholder for hardware offload.
Without a backend the device is software-only. See `offload.md`.

## Limitations

- Single instance.
- Software-only forwarding: IPTV unicast flows that used PPE L2B offload in
  the PON-native path are handled by the CPU/GRO path.
- Multicast forwarding is left to the bridge / IGMP proxy / multicast routing
  stack; no hardware port-mask offload.
- Only the basic VLAN demux is implemented. TPID/PCP/DEI, double tags and
  arbitrary vendor rules are out of scope for v1.
