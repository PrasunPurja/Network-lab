# addressing-scheme

## Scenario
Base network: `192.168.10.0/24`

| Requirement | Hosts Needed |
|---|---|
| Sales (VLAN 10) | 50 |
| HR (VLAN 20) | 20 |
| R1–R2 WAN link | 2 |

## Subnet Allocation (largest requirement first)

| Subnet | Mask | Usable Range | Broadcast |
|---|---|---|---|
| 192.168.10.0/26 (Sales) | /26 | .1 – .62 | .63 |
| 192.168.10.64/27 (HR) | /27 | .65 – .94 | .95 |
| 192.168.10.96/30 (WAN link) | /30 | .97 – .98 | .99 |

## Allocation Logic
Subnets were sized individually with VLSM rather than splitting the /24 into equal blocks,
to avoid wasting address space on the smaller requirements. Allocation was done largest-first:
Sales (needing the most hosts) was placed first at the start of the /24, HR was placed
next in the remaining space, and the WAN link — needing only 2 usable addresses — was given a
/30, the smallest practical subnet size, at the end. Each subnet starts immediately after the
previous subnet's broadcast address, with no overlap between ranges.
