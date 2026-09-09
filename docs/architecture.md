# Architecture

## Design goal

The lab models a headquarters and a remote branch connected through a routed WAN. The main constraint is that the remote FortiGate does **not** have a fixed WAN address; it learns its address by DHCP. HQ therefore cannot depend on a configured static branch gateway.

The solution uses HQ as an IKEv2/IPsec dynamic (dial-up) responder and the remote site as the initiator.

## Addressing plan

| Segment / node | Address | Purpose |
|---|---|---|
| HQ LAN | `192.168.10.0/24` | Headquarters private network |
| HQ FortiGate LAN | `192.168.10.1/24` | HQ default gateway |
| HQ FortiGate WAN | `10.0.20.2/24` | Static HQ VPN endpoint |
| HQ Alpine host | `192.168.10.60/24` | RDP client and test host |
| WAN transit | `10.0.12.0/30` | Router-to-router transit |
| Remote WAN | `10.0.30.0/24` | DHCP-enabled branch WAN |
| Remote FortiGate WAN | `10.0.30.100/24` observed | Dynamic VPN initiator |
| Remote LAN | `192.168.20.0/24` | Remote private network |
| Remote Windows | `192.168.20.100/24` | RDP target |
| Remote Linux | `192.168.20.10/24` | Auxiliary test host |

The DHCP address is an observed lab value, not an architectural dependency.

## Main components

- EVE-NG Community Edition
- 2 x FortiGate VM at HQ
- 1 x FortiGate VM at remote site
- 2 routed WAN devices
- Layer-2 switches at both sites
- Alpine Linux at HQ
- Windows 10 Pro at remote site
- auxiliary Linux endpoint

## HA design

HQ uses an active-passive FortiGate cluster:

- `port1`: HQ LAN
- `port2`: HQ WAN
- `port3`: HA heartbeat
- session pickup enabled
- primary member with higher priority
- secondary member with lower priority

Both members share the same LAN and WAN Layer-2 domains.

The lab's acceptance criteria were:

1. both members present and synchronized before failure;
2. the secondary becomes primary after the active unit stops;
3. inter-site traffic resumes;
4. the IPsec selector and remote route are restored.

## VPN design

### HQ

- role: dynamic/dial-up responder
- IKE version: IKEv2
- peer match: `REMOTE-SITE`
- route-based VPN
- `net-device disable`
- dynamic route installation enabled

### Remote

- role: initiator
- HQ gateway: `10.0.20.2`
- WAN: DHCP
- local IKE identity: `REMOTE-SITE`

### Protected subnets

- HQ -> Remote: `192.168.10.0/24` -> `192.168.20.0/24`
- Remote -> HQ: `192.168.20.0/24` -> `192.168.10.0/24`

## Traffic separation

Private inter-site traffic and Internet traffic use separate policies:

| Traffic | Path | NAT |
|---|---|---|
| HQ <-> Remote | LAN <-> IPsec | Disabled |
| LAN -> Internet | LAN -> WAN | Enabled |
| HA control | HQ FortiGate heartbeat link | N/A |

This keeps original private addresses intact inside the VPN while still allowing both sites to access the Internet independently.
