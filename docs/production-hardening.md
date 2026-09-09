# Production hardening

This repository documents the **validated lab state**, not a recommended production template.

## Lab state vs production direction

| Area | Lab state | Production direction |
|---|---|---|
| Cryptography | `des-sha256` because of the available VM image | Use a current supported FortiOS release and modern suites such as AES-GCM when supported |
| Peer authentication | Pre-shared key | Use unique managed secrets or certificate-based IKE authentication where appropriate |
| Firewall services | `ALL` for broad validation | Restrict source, destination, and services to business requirements and enable logging |
| HA heartbeat | One heartbeat link | Use redundant heartbeat paths where supported |
| IPsec HA | Basic failover validation | Validate current Fortinet HA/IPsec settings for the target release, including ESP sequence synchronization where supported |
| Monitoring | Local CLI and packet captures | Centralize VPN, firewall, and HA events in FortiAnalyzer/SIEM or equivalent |
| Management | Manual configuration | Use configuration backups, change tracking, review, and recovery/failback procedures |
| Scale | One branch | Evaluate dynamic routing, centralized management, ADVPN/SD-WAN, and structured route exchange |

## Cryptography

The lab used `des-sha256` because of the available FortiGate VM image. This is intentionally retained in the sanitized configuration so the repository remains faithful to what was actually tested.

Do **not** treat that proposal as a current recommendation.

Fortinet's current documentation exposes recommended IKE proposals based on modern AES, AES-GCM, and ChaCha20-Poly1305 options. NIST withdrew FIPS 46-3 (DES) in 2005 because DES no longer provided the security needed to protect information.

References:

- https://docs.fortinet.com/document/fortigate/latest/administration-guide/238852/encryption-algorithms
- https://csrc.nist.gov/pubs/fips/46-3/final
- https://csrc.nist.gov/pubs/sp/800/131/a/r2/final

## HA and IPsec

The lab enabled `session-pickup`, but a production design should be validated against the exact FortiOS release and hardware/platform behavior.

Current Fortinet documentation for IPsec in HA also identifies `ha-sync-esp-seqno` as a relevant Phase 1 setting for uninterrupted IPsec traffic during HA failover on supported releases.

Reference:

- https://docs.fortinet.com/document/fortigate/7.6.6/administration-guide/111309/ipsec-vpn-in-an-ha-environment

## Dynamic route installation

The dynamic-peer routing behavior used in this lab matches Fortinet's documented `add-route` behavior: the peer destination-selector route is installed when the dynamic tunnel negotiates.

Reference:

- https://docs.fortinet.com/document/fortigate/8.0.0/administration-guide/790613/phase-1-configuration

## Management access

The lab configurations include broad management access methods on interfaces because this was an isolated virtual environment. Production management should be restricted to dedicated management networks and required protocols only.

## Testing not performed

The project did not claim:

- production throughput benchmarks;
- long-duration availability tests;
- multi-WAN failover;
- PKI-based IKE authentication;
- centralized FortiManager/FortiAnalyzer management;
- large multi-branch scale;
- hitless IPsec failover.

Those remain future work rather than results of this lab.
