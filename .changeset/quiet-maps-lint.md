---
"kitcn": patch
---

## Patches

- Fix Next.js scaffolds to select and reconcile a compatible ESLint release.
- Fix cRPC client-error guards (`isCRPCClientError`, `isCRPCError`,
  `isCRPCErrorCode`) rejecting errors thrown through other entrypoints such as
  `kitcn/react` and `kitcn/solid`, so deterministic refusals like
  `UNAUTHORIZED` are no longer retried as transport failures.
