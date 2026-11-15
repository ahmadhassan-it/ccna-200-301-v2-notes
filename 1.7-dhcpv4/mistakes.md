# 1.7 DHCPv4: my mistakes

| # | Mistake | Truth |
|---|---|---|
| 1 | Said the Discover source is "broadcast" | Source is 0.0.0.0. The destination is the broadcast |
| 2 | `169.254` = "failed at Discover" | Discover was sent, but no Offer came back |
| 3 | Gave the fix but not the order | Exclusions first, then the pool |
| 4 | Wrote DSCP instead of DHCP | DSCP is QoS. DHCP gives addresses |
| 5 | Unclear answer about the helper on the wrong port | Helper goes where the client broadcast arrives |
| 6 | Removed the whole exclusion line | Replace it with a small range for the static devices |
| 7 | Typed `1P` (number one) instead of `ip` | Read the `^` marker. It points at the bad word |
| 8 | `default-router` missing in the pool | DHCP can work but give no gateway. Check all pool lines |
| 9 | No route on R2 back to 192.168.10.0/24 | The relay needs a return path too |
| 10 | Used `do lease 0 3 0` | `do` is for exec commands like `show`, not config commands |

Packet Tracer notes (not exam facts): `lease` and `show running-config interface` were not accepted on my router.
