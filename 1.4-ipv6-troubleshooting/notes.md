# 1.4 Troubleshoot IPv6 addressing (CCNA 200-301 v2.0)

Cisco v2.0 topic 1.4: find faults in IPv6 address configuration, assignment and prefix size, for unicast addresses and modified EUI-64.

## Contents

1. [Why IPv6](#1-why-ipv6)
2. [Address format and shortening](#2-address-format-and-shortening)
3. [Prefix: network part and device part](#3-prefix-network-part-and-device-part)
4. [Address types](#4-address-types)
5. [Router commands](#5-router-commands)
6. [Modified EUI-64](#6-modified-eui-64)
7. [Troubleshooting](#7-troubleshooting)
8. [Packet Tracer lab](#8-packet-tracer-lab)
9. [Must memorize](#9-must-memorize)
10. [Mistake log](#10-mistake-log)
11. [Word list](#11-word-list)

---

## 1. Why IPv6

```
IPv4   32 bits   ->  about 4.3 billion addresses   (ran out)
IPv6   128 bits  ->  about 3.4 x 10^38 addresses
```

## 2. Address format and shortening

8 groups x 4 hex digits x 4 bits = 128 bits. Hex digits are `0-9` and `A-F` (no letter O).

```
2001:0DB8:0000:0000:0000:0000:0A00:0010
 (1)  (2)  (3)  (4)  (5)  (6)  (7)  (8)
```

- Rule 1: drop the zeros at the start of each group (`0A00` becomes `A00`, never `A`).
- Rule 2: replace ONE run of all-zero groups with `::`. Only one `::` per address.
- Two zero runs: squeeze the longest. If equal, squeeze the first.
- Hidden groups = 8 minus the groups you can see.

```
2001:0DB8:0000:0000:0000:0000:0A00:0010
2001:DB8:0:0:0:0:A00:10        (Rule 1)
2001:DB8::A00:10               (Rule 2)
```

Broken addresses: two `::`, a letter G, 9 groups, 5 digits in one group.

## 3. Prefix: network part and device part

The number after `/` = how many bits at the start are the network.

```
/64:   groups 1-4 = network (prefix)  |  groups 5-8 = device (interface ID)
2001:0DB8:ACAD:0005 | 0000:0000:0000:0010
```

- Network of a /64 address: keep the first 4 groups, then `::/64`.
  `2001:DB8:ACAD:5:A1B2:C3D4:E5F6:789/64` is in `2001:DB8:ACAD:5::/64`
- Other prefix lengths: hex digits to keep = prefix / 4, the rest become zeros.
  `2001:DB8:ACAD:5A3C::1/56` is in `2001:DB8:ACAD:5A00::/56`
- Always write a network with `::/prefix`.
- /64 LANs inside a bigger block = 2^(64 - prefix). A /48 gives 65,536. A /52 gives 4,096.
- A /48 from the ISP: only group 4 changes for each LAN (`...:1::/64`, `...:2::/64`, `...:A::/64`). It is hex, so after 9 comes A.
- A LAN is always /64. SLAAC needs it.

## 4. Address types

```
Type             Starts with            Works where
Global unicast   2000::/3 (2 or 3)      Internet
Unique local     FC00::/7 (FD in use)   One company only
Link-local       FE80::/10              One link only
Loopback         ::1/128                The device itself
Unspecified      ::                     "I have no address yet"
```

- Every IPv6 interface has a link-local address once IPv6 is on.
- A router does not forward packets with a link-local source or destination to another network.
- FE80 is NOT like private IPv4. The private-like type is unique local (FD...).
- Extra, one line each: multicast starts with FF (`FF02::1` all devices on the link, `FF02::2` all routers on the link). IPv6 has no broadcast. Anycast = the same address on several devices, the nearest one answers.

## 5. Router commands

```
Router(config)# ipv6 unicast-routing
Router(config)# interface gigabitethernet0/0
Router(config-if)# ipv6 address 2001:DB8:ACAD:1::1/64
Router(config-if)# ipv6 address FE80::1 link-local
Router(config-if)# no shutdown
```

| Command | What it does |
|---|---|
| `ipv6 unicast-routing` (global) | Router forwards IPv6 between networks. Without it, no forwarding. |
| `ipv6 address X/64` | Full address by hand. No space before `/64`. |
| `ipv6 address X/64 eui-64` | Type only the prefix. Router builds the last 64 bits from the MAC. |
| `ipv6 address FE80::1 link-local` | Your own link-local. The word `link-local` is needed. |
| `ipv6 enable` | Makes a link-local only. |
| `ipv6 address autoconfig` | Router builds its address from a Router Advertisement (SLAAC). |

SLAAC (Stateless Address Autoconfiguration): a device builds its own address from the prefix a router tells it.

```
PC  --- RS: "Is any router here?" --->  Router
PC  <-- RA: "Prefix is X/64"      ----  Router
PC builds: prefix from RA + its own 64-bit interface ID
```

Show commands: `show ipv6 interface brief`, `show ipv6 interface g0/0`, `show ipv6 route`, `show ipv6 neighbors` (NDP table, the IPv6 version of ARP).

## 6. Modified EUI-64

Build a 64-bit interface ID from the 48-bit MAC: cut, FFFE, flip.

```
MAC 0040.0b80.4001  =  00:40:0B:80:40:01

Job 1  cut in the middle:      00:40:0B   |   80:40:01
Job 2  insert FF:FE:           00:40:0B : FF:FE : 80:40:01
Job 3  flip the 7th bit:       00 -> 02

Interface ID: 0240:0BFF:FE80:4001   (short: 240:BFF:FE80:4001)
```

Flip table (second digit of the first byte, works both ways):

```
0 <-> 2    1 <-> 3    4 <-> 6    5 <-> 7
8 <-> A    9 <-> B    C <-> E    D <-> F
```

Full address = prefix + interface ID:
`2001:DB8:ACAD:2` + `240:BFF:FE80:4001` = `2001:DB8:ACAD:2:240:BFF:FE80:4001/64`

- `[EUI]` at the end of an address in `show ipv6 interface` means it was made with EUI-64.
- `FF:FE` in the middle of the last 64 bits = made from a MAC.
- The automatic link-local also uses the MAC this way.

## 7. Troubleshooting

```
1. Interface up/up?                  show ipv6 interface brief
2. Same first 4 groups? (/64)        compare router and PC
3. Gateway on the PC's link?         see the rule below
4. ipv6 unicast-routing on?          show running-config
5. Routes?                           show ipv6 route
```

Compare ONLY the first 4 groups for /64. The end part is supposed to be different.

Gateway rule: a PC's gateway is valid if it is
- a global address inside the PC's own /64, or
- the router's FE80 address on that same link (the normal way when a PC learns its gateway from a router).

It is wrong if it is a global address from a different /64.

| What I see | Likely fault |
|---|---|
| administratively down | Missing `no shutdown` |
| PC pings router, not other networks | Wrong gateway, or no `ipv6 unicast-routing` |
| PC cannot ping the router, both look right | Different group 4 |
| Command rejected | Double `::`, letter O, or 9 groups |
| Behaves like another network | Wrong prefix length |

A ping needs a route both ways. The reply must also find its way back, so the far device needs a route to the sender's network.

## 8. Packet Tracer lab

```
PC1 ---- SW1 ---- R1 ---- R2
LAN 1: 2001:DB8:ACAD:1::/64     Link: 2001:DB8:ACAD:2::/64
```

R1:
```
ipv6 unicast-routing
interface GigabitEthernet0/0
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:1::1/64
interface GigabitEthernet0/1
 ipv6 address 2001:DB8:ACAD:2::1/64
```

R2:
```
ipv6 unicast-routing
interface GigabitEthernet0/0
 ipv6 address 2001:DB8:ACAD:2::/64 eui-64
```

PC1: `2001:DB8:ACAD:1::10`, gateway `FE80::1`.

R2 output:
```
R2# show ipv6 interface brief
GigabitEthernet0/0   [up/up]
    FE80::240:BFF:FE80:4001
    2001:DB8:ACAD:2:240:BFF:FE80:4001

R2# show ipv6 route
C   2001:DB8:ACAD:2::/64
L   2001:DB8:ACAD:2:240:BFF:FE80:4001/128
L   FF00::/8
```

- I worked out R2's interface ID by hand from MAC `0040.0b80.4001`. It matches what the router built.
- R2's routing table has only its own link. It has no route to LAN 1.
- Ping PC1 to R2's EUI-64 address (2001:DB8:ACAD:2:240:BFF:FE80:4001): 100% loss, 4 requests timed out. R2's routing table has no route to LAN 1, which fits this result. A static route on R2 would fix it (topic 3.2).

## 9. Must memorize

```
128 bits = 8 groups x 4 hex digits. One :: only. No broadcast. No letter O.
/64 = LAN.  /48 = site (common).  /128 = one address.
Digits to keep = prefix / 4.   LANs in a block = 2^(64 - prefix).
2 or 3 = global | FC/FD = unique local | FE80 = link-local | ::1 = loopback
EUI-64: cut -> FFFE -> flip 7th bit.   Full address = prefix + ID.
ipv6 unicast-routing | ipv6 address X/64 [eui-64] | ipv6 address FE80::1 link-local
show ipv6 interface brief | show ipv6 route | show ipv6 neighbors
C = connected network    L = local address (/128)
```

## 10. Mistake log

1. Typed the letter O instead of the number zero in a hex address.
2. Gave only the interface ID when the question asked for the full address.
3. Changed group 3 instead of group 4 when fixing a wrong prefix.
4. Compared the end of the addresses (`::1` and `::20`) instead of the first 4 groups.
5. Wrote a network without `::/prefix`.
6. Said routing protocols drop FE80 packets. Really, the router does not forward link-local packets.

## 11. Word list

- Bit: one 0 or one 1. Byte: 8 bits.
- Hex: base 16 (`0-9`, `A-F`).
- Prefix: the network part at the start of the address.
- Interface ID: the device part, the last 64 bits in a /64.
- MAC address: hardware address of a network card, 48 bits.
- ISP: the company that gives Internet.
- LAN: one local network.
- Unicast: one sender to one receiver.
- Link: one connected network segment.
- Link-local: address valid on one link only (FE80).
- Gateway: the router that leads out of the LAN.
- NDP: Neighbor Discovery Protocol, does the job of ARP in IPv6.
- SLAAC: a device builds its own address from a router's prefix.
- RS / RA: Router Solicitation (question) and Router Advertisement (answer).

Commands and facts checked against the Cisco v2.0 exam topic list and Cisco IOS behaviour. Verify important commands in Jeremy's IT Lab videos too.
