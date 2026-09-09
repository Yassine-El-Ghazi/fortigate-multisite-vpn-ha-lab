# Configuration files

These files are sanitized from the lab artifacts used during the project.

## Important

- PSKs and HA passwords are replaced with `<REDACTED>`.
- No serial numbers or licensing identifiers are included.
- `des-sha256` is preserved because it reflects the validated lab state. It is **not** a production recommendation.
- Broad `service ALL` policies reflect validation scope, not a least-privilege production policy.
- `auto-asic-offload disable` appears in the final lab FortiGate files because it was used during troubleshooting. It was not the routing fix.
- The files are intended as reproducible lab references, not drop-in production configurations.

See [production-hardening.md](../docs/production-hardening.md).
