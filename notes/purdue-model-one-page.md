# The Purdue model in one page

A quick reference for the reference architecture that underpins most OT network and IEC 62443
segmentation discussions. The point of the model is not the levels for their own sake — it's that
**traffic should flow vertically through boundaries you control, not sideways across them.**

## The levels

| Level | Name | Typical assets | Talks to |
|---|---|---|---|
| **5** | Enterprise | Internet, corporate services | L4 |
| **4** | Enterprise / business | ERP, business clients, email | L3.5 (via DMZ) |
| **3.5** | **Industrial DMZ** | historian replica, patch/AV relay, jump host | brokers L4 ↔ L3 |
| **3** | Operations / site | historian, MES, OT domain controller, OT servers | L2, L3.5 |
| **2** | Supervisory control | SCADA, HMI, engineering workstation | L1, L3 |
| **1** | Basic control | PLC, RTU, IED | L0, L2 |
| **0** | Process | sensors, actuators, drives, field I/O | L1 |

## The rules that actually matter

1. **No direct Enterprise ↔ Control traffic.** Level 4 must not talk straight to Level 1/2. It goes
   through the **Level 3.5 industrial DMZ**, which is the single brokered crossing point.
2. **The DMZ breaks the path.** Enterprise pushes to a DMZ host; OT pulls from a DMZ host. Nothing
   initiates a session straight through. A historian *replica* in the DMZ is the classic example —
   the real historian (L3) replicates one-way into the DMZ, and enterprise reads the replica.
3. **Each boundary is a conduit** (in IEC 62443 terms) with a firewall, an allow-list of exactly the
   approved flows, and a default-deny. The "allowed flows" list is the single source of truth — the
   diagram, the firewall rules and the FR5 evidence all come from it.
4. **Safety is its own zone.** A SIS / safety PLC is kept separate, usually monitored (read-only)
   rather than reachable for control from the supervisory zone.

## Where it bends in real life

- Purely flat plant networks still exist; the model is the *target*, and the first win is usually
  just getting a real DMZ in between enterprise and OT.
- Remote access (vendors, support) should land in the DMZ via a jump host, never straight into L2.
- "Level 3.5" is a convention, not a law — some sites call it the OT-DMZ or the perimeter zone.

## Related

- [Why GOOSE stops at the control boundary and IEC 104 goes north](goose-vs-iec104-boundary.md)
- The `iec62443-zone-kit` tool turns an asset list + allowed-flows into exactly this zoning, with a
  firewall matrix and an FR1–FR7 checklist.
