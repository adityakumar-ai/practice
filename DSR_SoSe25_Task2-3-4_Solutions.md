# DSR SoSe 2025 Exam — Full Solutions & Explanations
### Task 2 (IPv4 / OSPF / BGP), Task 3 (IPv6), Task 4 (SDN Flow Tables)

Everything below is worked out **from the evidence actually given on the exam** (routing tables, traceroute output, the BGP capture, the RA capture) — not guesswork. Wherever a value was cross-checked against two or more independent pieces of evidence, I've said so, so you know how solid it is.

---

## 🧩 TASK 2 — IPv4 Addressing, OSPF, BGP

### The big picture (explained like you're 5)

Imagine two office buildings (two **Autonomous Systems**, AS 6500 and AS 1900). Inside each building there are hallways (OSPF Areas) connecting rooms (routers). Between the two buildings there's one single phone line (the **BGP** link between R5 and R6) — that's the *only* door connecting the two companies' networks.

Every hallway segment between two routers is its own tiny network with only **2 usable addresses** (one for each router) — these are called **/30 networks** (see the box below for why). Every "room" where a PC sits (pc1, pc2, pc3, pc4) is a **/24 network** — a much bigger hallway that can fit 254 computers.

**Why is a /30 only 2 usable IPs?**
A /30 mask means 30 bits are "network" and only 2 bits are left for "host". 2 bits = 2² = 4 total addresses. But the *first* address of any subnet is always the "network address" (can't be assigned to a device) and the *last* one is always the "broadcast address" (means "everyone on this subnet"). So 4 − 2 = **2 usable addresses**. That's exactly enough for a point-to-point link between two routers — no more, no less.

Example: `192.168.10.4/30` → addresses are `.4 (network), .5, .6, .7 (broadcast)`. Only `.5` and `.6` can be given to routers.

### How I found every address

The exam gives you 3 gold-mine clues, and if you combine them you can rebuild the *whole* map:

1. **R2's routing table** — tells you which /30 subnets exist and which router-IP is the "next hop" to reach things beyond a neighbor.
2. **R7's routing table** — same, but for the other AS.
3. **The traceroute from pc1 to pc3 (Fig 2.4)** — this is the best clue of all, because a traceroute *literally lists every router IP along the path, in order*. It's like footprints in the snow.
4. **The BGP capture at R5 (Fig 2.5)** — tells you the exact IPs of the two ends of the R5–R6 phone line.

Reading the traceroute (pc1 → pc3, i.e. 200.130.26.2):
```
1  200.130.24.1     ← R0 (the door out of pc1's room)
2  192.168.10.6     ← R2 (next room over)
3  192.168.10.14    ← R3 (next room)
4  192.168.10.18    ← R4 (next room)
5  200.130.26.2     ← pc3 itself (arrived!)
```
This single traceroute confirms 4 router addresses at once, and also tells us there must be a small hallway between R3 and R4 that isn't shown in R2's table (because R2 doesn't need to know about it directly — it just knows "send anything for pc3 to R3, R3 will handle the rest").

There's one more trick used throughout this exam: **the router with the lower number gets the lower address on each link** (R0 gets a lower number than R2 on the R0–R2 link, R2 gets lower than R3 on the R2–R3 link, and so on). This pattern holds consistently for every link we can double-check, so it's safe to use it to fill in the couple of links we don't have direct traceroute proof for.

### ✅ Task 2(a) — IPv4 addresses of specific interfaces

| Interface | Address | How we know |
|---|---|---|
| R0, eth0 | **200.130.24.1** | Traceroute hop 1 (pc1's default gateway) |
| R0, eth2 | **192.168.10.5** | R2's table: next-hop to reach R0-side networks |
| R1, eth1 | **192.168.10.9** | R2's table: next-hop to reach pc2's network |
| R2, eth0 | **192.168.10.6** | Traceroute hop 2 |
| R3, eth0 | **192.168.10.14** | Traceroute hop 3 (also R2's table) |
| R4, eth1 | **192.168.10.18** | Traceroute hop 4 |
| pc3, eth0 | **200.130.26.2** | Traceroute destination |
| R5, eth1 | **192.168.10.25** | BGP capture (Fig 2.5): this packet was *received at* R5, so the destination address, `192.168.10.25`, is R5's own address |
| R6, eth1 | **192.168.10.26** | BGP capture: the source address is R6's (the other end of the same phone line) |
| R6, eth0 | **192.168.10.29** | R7's table: next-hop to reach anything beyond R6 |
| R8, eth1 | **192.168.10.34** | R7's table: next-hop to reach pc4's network |

### ✅ Task 2(b) — Network prefixes

| Link | Prefix |
|---|---|
| pc1–R0 | **200.130.24.0/24** |
| pc2–R1 | **200.130.25.0/24** |
| R0–R1 | **192.168.10.0/30** |
| R0–R2 | **192.168.10.4/30** |
| R1–R2 | **192.168.10.8/30** |
| R2–R3 | **192.168.10.12/30** |
| R4–pc3 | **200.130.26.0/24** |
| R3–R5 | **192.168.10.20/30** |
| R6–R7 | **192.168.10.28/30** |
| R7–R8 | **192.168.10.32/30** |
| R8–pc4 | **200.130.27.0/24** |

Notice the pattern: 4 addresses are "used up" per router-to-router link (`.0/.4/.8/.12/.16/.20/.24/.28/.32...`), even though only 2 are usable — that's just the price of using /30s, and it's completely normal.

---

## 🧩 TASK 3 — IPv6 Addressing & Autoconfiguration

### The big picture (explained like you're 5)

IPv6 addresses are long (128 bits), but they're built from two Lego pieces glued together:

```
[ 64-bit NETWORK PREFIX ]  +  [ 64-bit INTERFACE ID ]
```

- The **prefix** (first half) says *which room/hallway* you're in — this is like the /24 or /30 in IPv4, except it's *always* 64 bits for a normal subnet.
- The **Interface ID** (second half) says *which specific device* you are — a bit like your seat number.

A **link-local address** (`fe80::/64` prefix) is a "local only, don't leave this room" address — every interface always has one automatically, it's never routed anywhere else.
A **global unicast address** uses the *same* Interface ID, but swaps `fe80::` for a real routable prefix (like `2001:7::/64`). So: once you know a device's Interface ID (from its link-local address), you instantly know its global address too, on any subnet whose prefix you know — just glue the same tail onto a different prefix.

### How autoconfiguration (SLAAC) builds the Interface ID from a MAC address

This is the "EUI-64" trick, and it's simpler than it looks:

1. Take the 6-byte MAC address, e.g. `00:00:00:AA:00:0F`
2. Split it in half: `00:00:00` and `AA:00:0F`
3. Stuff `FF:FE` in the middle: `00:00:00:FF:FE:AA:00:0F`
4. Flip the 7th bit of the very first byte (the "universal/local" bit — this just tells the world "I made this address up myself locally"). `00` in binary is `00000000` → flipping bit 7 gives `00000010` = `02`.
5. Result: `02:00:00:FF:FE:AA:00:0F` → written in IPv6 hex-group style: `0200:00ff:feaa:000f` → shortened: `200:ff:feaa:f`

So MAC `00:00:00:aa:00:0f` becomes Interface ID `::200:ff:feaa:f`.
**You can also run this backwards** (which is exactly what part (a) asks you to do): given the address, strip the `fe80::` prefix, remove the `ff:fe` in the middle, and un-flip that same bit.

### ✅ Task 3 Answers

**a) MAC address of n9**
Its link-local address is `fe80::200:ff:feaa:f`. Reversing EUI-64 (remove `ff:fe`, flip the bit back) gives:
**MAC = 00:00:00:aa:00:0f**

**b) What does the ICMP message in Fig 3.2 address?**
It's sent to `ff02::16`. This is a *reserved, well-known* multicast address that always means:
**All MLDv2-capable routers on the local link.** (n9 is announcing "I want to join a multicast group" and every router that understands MLDv2 needs to hear it.)

**c) What does the ICMP message in Fig 3.3 address?**
It's sent to `ff02::1`. This is the other famous reserved multicast address, meaning:
**All nodes on the local link** (every host and router, no exceptions) — this is how a Router Advertisement reaches everyone at once.

**d) Link-local unicast address of n9**
Same Interface ID we found in (a), with the link-local prefix:
**fe80::200:ff:feaa:f**

**e) Global unicast address of n9**
n5's Router Advertisement (Fig 3.3) tells n9 which prefix to use for the *real, routable* address: `2001:7::/64`. Glue on the same Interface ID as before:
**2001:7::200:ff:feaa:f**

**f) Network prefixes of each link**
(Confirmed two independent ways: n3's routing table lists exactly the subnets `2001:1::/64` through `2001:11::/64` once each, and the traceroute in Fig 3.4 directly shows 4 of them in a row — a great double-check.)

| Link | Prefix |
|---|---|
| n7–n1 | **2001:10::/64** |
| n1–n2 | **2001:11::/64** |
| n1–n3 | **2001:1::/64** |
| n2–n3 | **2001:6::/64** |
| n2–n4 | **2001:2::/64** |
| n3–n4 | **2001:4::/64** |
| n3–n5 | **2001:5::/64** |
| n4–n6 | **2001:3::/64** |
| n5–n6 | **2001:9::/64** |
| n6–n8 | **2001:8::/64** |
| n5–n9 | **2001:7::/64** |

*(Sanity check: the traceroute n7→n8 goes `2001:10::1 → 2001:11::2 → 2001:2::2 → 2001:3::2 → 2001:8::20`, i.e. hop-by-hop through n1→n2→n4→n6 — every single one of those prefixes matches the table above. That's about as confirmed as it gets.)*

**g) Link-local unicast addresses on specific interfaces**
n3's own routing table shows its 4 own interface addresses directly (one per port), and Fig 3.3 gives us n5's:

| Interface | Link-local address |
|---|---|
| n3-eth0 (→ n1) | **fe80::200:ff:feaa:3** |
| n3-eth1 (→ n5) | **fe80::200:ff:feaa:8** |
| n3-eth2 (→ n2) | **fe80::200:ff:feaa:a** |
| n3-eth3 (→ n4) | **fe80::200:ff:feaa:d** |
| n5-eth1 (→ n9) | **fe80::200:ff:feaa:e** *(this is literally the source address in Fig 3.3 — n5 sending its own RA)* |
| n9-eth0 | **fe80::200:ff:feaa:f** *(same as part d)* |

**h) Global unicast addresses on specific interfaces**
Once you know an interface's Interface ID (from its link-local address, or from a traceroute hop) and the prefix of the link it's on (from part f), just glue them together:

| Interface | Global address | Why |
|---|---|---|
| n7-eth0 | **2001:10::20** | traceroute source address |
| n1-eth2 (→ n7) | **2001:10::1** | traceroute hop 1 |
| n2-eth0 (→ n1) | **2001:11::2** | traceroute hop 2 |
| n4-eth0 (→ n2) | **2001:2::2** | traceroute hop 3 |
| n6-eth0 (→ n4) | **2001:3::2** | traceroute hop 4 |
| n8-eth0 | **2001:8::20** | traceroute destination |
| n3-eth0 (→ n1) | **2001:1::200:ff:feaa:3** | prefix (n1–n3 link) + n3's own Interface ID on that port |
| n3-eth1 (→ n5) | **2001:5::200:ff:feaa:8** | prefix (n3–n5 link) + Interface ID |
| n3-eth2 (→ n2) | **2001:6::200:ff:feaa:a** | prefix (n2–n3 link) + Interface ID |
| n3-eth3 (→ n4) | **2001:4::200:ff:feaa:d** | prefix (n3–n4 link) + Interface ID |

---

## 🧩 TASK 4 — SDN Flow Tables

### The big picture (explained like you're 5)

Forget "learning" like a normal Ethernet switch does — an **SDN switch has no brain of its own**. It's a dumb, obedient forwarding table that only does exactly what the **Controller** tells it to do: *"if a packet comes in on this port, going from this network to that network, send it out that port."* Nothing more.

Your job in this task is to be the Controller: figure out the shortest path between two networks, then write down, **for every switch that the traffic actually passes through**, one row for the forward direction and one row for the reverse direction (since replies need to get back too!).

**The topology** (5 links total among 4 switches):
```
        Net1                      Net3
         |                          |
   [Sw1]P10          P13——P31  [Sw3]P30
      P12|                P32|    |P34
         |                    \    |
   [Sw2]P21——(direct)          \   |
      P20|  \                   \  |
         |   P23————————————————(same link, other end)
       Net2  P24——P42        [Sw4]P43
              |                |P40
           (to Sw4)          Net4
```
In words: **Sw1** only touches Sw2 and Sw3. **Sw4** only touches Sw2 and Sw3. Sw2 and Sw3 touch everything. There is **no direct Sw1–Sw4 link**.

### Task 4(a) — Route Net1↔Net3 and Net2↔Net3 (shortest path)

- Net1→Net3: Sw1 and Sw3 are directly connected (P13–P31) → 1 hop, done.
- Net2→Net3: Sw2 and Sw3 are directly connected (P23–P32) → 1 hop, done.

| Switch | Port in | Src Net | Dst Net | Port out |
|---|---|---|---|---|
| **Sw1** | P10 | Net1 | Net3 | P13 |
| Sw1 | P13 | Net3 | Net1 | P10 |
| **Sw2** | P20 | Net2 | Net3 | P23 |
| Sw2 | P23 | Net3 | Net2 | P20 |
| **Sw3** | P31 | Net1 | Net3 | P30 |
| Sw3 | P30 | Net3 | Net1 | P31 |
| Sw3 | P32 | Net2 | Net3 | P30 |
| Sw3 | P30 | Net3 | Net2 | P32 |
| **Sw4** | — | — | — | — |

(Sw3 needs *two* pairs of rows because traffic from two different sources — Net1 and Net2 — both arrive at Sw3 wanting to reach Net3, but from two different incoming ports, and traffic leaving Sw3 toward Net3 needs to know which way to send the *reply*.)

Sw4 is untouched: this traffic never goes near it, so every entry stays empty.

### Task 4(b) — Also add Net2↔Net4

Sw2 and Sw4 are directly connected (P24–P42) → 1 hop.

| Switch | Port in | Src Net | Dst Net | Port out |
|---|---|---|---|---|
| **Sw2** | P20 | Net2 | Net4 | P24 |
| Sw2 | P24 | Net4 | Net2 | P20 |
| **Sw4** | P42 | Net2 | Net4 | P40 |
| Sw4 | P40 | Net4 | Net2 | P42 |

Sw1 and Sw3 are untouched (this traffic never passes through them).

### Task 4(c) — Link P13–P31 fails; route only Net1↔Net3 and Net2↔Net4

Losing P13–P31 removes Sw1's only direct road to Sw3. Since Sw1 only touches Sw2 and Sw3, the *only* remaining way from Sw1 to Sw3 is: **Sw1 → Sw2 → Sw3** (via P12–P21, then P23–P32). Net2↔Net4 is unaffected (its link P24–P42 is fine), so it stays exactly as in part (b).

| Switch | Port in | Src Net | Dst Net | Port out |
|---|---|---|---|---|
| **Sw1** | P10 | Net1 | Net3 | **P12** *(was P13)* |
| Sw1 | **P12** | Net3 | Net1 | P10 |
| **Sw2** | **P21** | Net1 | Net3 | **P23** *(new: Sw2 now relays this traffic)* |
| Sw2 | **P23** | Net3 | Net1 | **P21** |
| Sw2 | P20 | Net2 | Net4 | P24 *(unchanged)* |
| Sw2 | P24 | Net4 | Net2 | P20 *(unchanged)* |
| **Sw3** | **P32** | Net1 | Net3 | P30 *(was via P31)* |
| Sw3 | P30 | Net3 | Net1 | **P32** |
| **Sw4** | P42 | Net2 | Net4 | P40 *(unchanged)* |
| Sw4 | P40 | Net4 | Net2 | P42 *(unchanged)* |

### Task 4(d) — Add bandwidth limits: Net1↔Net3 ≈ 200 Mbit/s, Net2↔Net4 ≈ 500 Mbit/s, link P23–P32 only 100 Mbit/s (everything else 1 Gbit/s)

**The trap:** the path we just built in (c) for Net1–Net3 goes *straight through P23–P32* — but that link can only carry 100 Mbit/s, and Net1–Net3 needs 200 Mbit/s. It simply won't fit. Fewest-hops is no longer good enough; the Controller has to pick a path with **enough spare capacity**, even if it's longer.

Is there another way from Sw2 to Sw3 that avoids P23–P32? Yes — via Sw4: **Sw2 → Sw4 → Sw3** (using P24–P42, then P43–P34), both of which are 1 Gbit/s links. Checking the numbers: the Sw2–Sw4 link already carries 500 Mbit/s of Net2–Net4 traffic; adding another 200 Mbit/s of Net1–Net3 traffic gives 700 Mbit/s — well under the 1 Gbit/s ceiling. The Sw4–Sw3 link is completely free, so 200 Mbit/s fits easily. Problem solved by rerouting, not by upgrading any link.

New path for Net1↔Net3: **Sw1 → Sw2 → Sw4 → Sw3** (longer, but it actually works).
Net2↔Net4 traffic keeps using its direct 1 Gbit/s link, unchanged.

| Switch | Port in | Src Net | Dst Net | Port out |
|---|---|---|---|---|
| **Sw1** | P10 | Net1 | Net3 | P12 *(unchanged — still heads to Sw2 first)* |
| Sw1 | P12 | Net3 | Net1 | P10 |
| **Sw2** | P21 | Net1 | Net3 | **P24** *(changed! now sends toward Sw4, not Sw3)* |
| Sw2 | **P24** | Net3 | Net1 | P21 |
| Sw2 | P20 | Net2 | Net4 | P24 *(unchanged)* |
| Sw2 | P24 | Net4 | Net2 | P20 *(unchanged)* |
| **Sw3** | **P34** | Net1 | Net3 | P30 *(now arrives from Sw4, not Sw2)* |
| Sw3 | P30 | Net3 | Net1 | **P34** |
| **Sw4** | **P42** | Net1 | Net3 | **P43** *(new: Sw4 becomes a relay for this flow)* |
| Sw4 | **P43** | Net3 | Net1 | **P42** |
| Sw4 | P42 | Net2 | Net4 | P40 *(unchanged)* |
| Sw4 | P40 | Net4 | Net2 | P42 *(unchanged)* |

*(Note: Sw2 and Sw4 now each have two different rows that share the same physical port number, e.g. "port in = P24" — that's fine! A flow-table entry is matched on the **combination** of port + source network + destination network, not the port alone, so two flows can quietly share a cable without ever getting confused.)*
