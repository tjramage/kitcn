---
"kitcn": patch
---

## Patches

- Fix cRPC client-error guards (`isCRPCClientError`, `isCRPCError`,
  `isCRPCErrorCode`) rejecting errors thrown through other entrypoints such as
  `kitcn/react` and `kitcn/solid`, so deterministic refusals like
  `UNAUTHORIZED` are no longer retried as transport failures.
- Fix spurious internal errors during sessionless social sign-in redirects
  while preserving Convex JWT cookie issuance for authenticated sessions.
