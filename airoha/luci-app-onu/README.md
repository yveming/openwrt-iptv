# luci-app-onu IPTV files (extracted)

These files are extracted from `luci-app-onu` in the HG5582A tree at the
commit recorded in `../MANIFEST.md`. They are not a standalone package.

| File | Role |
| --- | --- |
| `files/etc/config/onu-iptv` | UCI defaults for the IPTV page |
| `files/etc/init.d/iptv` | Boot-time apply service |
| `files/usr/libexec/iptv` | Backend that applies the IPTV topology |
| `files/usr/sbin/iptv-ppe` | PPE L2B diagnostic helper |

To reproduce the archived variant, patch these files into the matching
`luci-app-onu` base instead of installing them as a separate package.
