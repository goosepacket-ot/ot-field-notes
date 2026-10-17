# Why GOOSE stops at the control boundary and IEC 104 goes north

A question that comes up constantly when people first read an IEC 61850 substation diagram:
*if GOOSE already carries the trip and status between IEDs, why not just use it all the way up to
the SCADA / control centre? Why switch to IEC 60870-5-104 at the station boundary?*

The short answer: **GOOSE and 104 solve different problems, at different layers, over different
networks.** They are not interchangeable.

## GOOSE — horizontal, fast, local

- **Layer 2 Ethernet multicast** (EtherType `0x88B8`). No IP, no TCP — it never routes.
- Built for **protection-class speed**: a trip must publish in a few milliseconds. It achieves that
  by re-transmitting with a shrinking interval and carrying a `stNum`/`sqNum` so subscribers detect
  loss instantly.
- It is **publisher→subscribers** inside a station bus. Every subscriber on that VLAN sees it.
- Because it is L2 multicast, it is **bounded to the station LAN by design**. You do not want trip
  messages flooding across a WAN, and they could not be routed there anyway.

So GOOSE is the *horizontal* language between bay-level IEDs and the station bus — bay-to-bay
interlocking, bus protection, breaker fail.

## IEC 104 — vertical, routable, supervisory

- **TCP/IP on port 2404** — routable across a WAN to the control centre.
- Built for **supervisory telemetry and control**: measurements, status, commands, with timestamps
  and quality bits, over a link that may have latency and may cross routers and firewalls.
- It is a **client/server** (controlling station ↔ controlled station) session with STARTDT/STOPDT,
  interrogation and sequence numbers for reliable delivery.

So 104 is the *vertical* language from the station up to the SCADA master.

## Why the switch happens at the gateway

The station **gateway / RTU** is the translation point. It subscribes to the GOOSE and polled data
inside the station, and re-presents the relevant points northbound as 104 (or DNP3, or MMS). That
boundary exists because:

| | GOOSE | IEC 104 |
|---|---|---|
| Layer | L2 multicast | L3 TCP/IP |
| Scope | station LAN only | routable WAN |
| Purpose | protection / interlock (ms) | supervisory telemetry/control |
| Delivery | repeat + sequence | TCP + APCI sequence |
| Crosses a router? | no | yes |

Pushing GOOSE northbound would mean L2-stretching a protection network across a WAN — a bad idea for
both performance and security. Pulling 104 down to the bay level would be too slow for protection.
Each protocol stays in the layer it was built for, and the gateway bridges them.

## The security angle (why this matters for segmentation)

This boundary is also a natural **zone boundary** (IEC 62443):
- The station bus (GOOSE/SV/MMS) is one zone — high integrity, low latency, no routing out.
- The northbound 104 link is a **conduit** to the control-centre zone, and it is exactly where you
  apply firewalling, allow-listing of the controlling station's IP, and (where supported) IEC
  62351 TLS. GOOSE itself has no authentication without IEC 62351-6, which is another reason to keep
  it contained.

## Takeaway

GOOSE = horizontal, L2, protection-speed, station-local.
IEC 104 = vertical, L3, supervisory, routable.
The gateway is the deliberate seam between them — and a good place to draw a security zone boundary.
