# Global Review

This review fixes the system-level guarantees that connect deposit, withdraw, and the surrounding config surface into one bridge security model.

## Global Invariants

- destination-side asset credit/release must not happen without source-side accounting
- asset movement must happen only through the intended bridge lifecycle paths
- L1/L2 token correspondence must remain aligned across the whole bridge lifecycle
- disabled hook / extra-data branches must not silently change normal bridge semantics
- the router-level routing model must not diverge from the gateway-level handling model
- counterpart-gated finalization must remain mandatory for destination-side asset movement
- recovery / fallback branches must not conflict with the normal branch
- the privileged / config / init / upgrade surface must not silently break bridge guarantees
