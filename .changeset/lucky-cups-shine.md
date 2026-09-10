---
"kitcn": patch
---

## Patches

- Fix `CRPCProvider` clearing authenticated cRPC queries when Convex confirms a
  token the client already held. The automatic reset compared `isAuthenticated`
  as well as the token identity. That flag turns from false to true on every
  server-rendered page once Convex validates the token in the HTML, so the
  provider dropped hydrated data on load and the screen lost the rows it had
  rendered. The reset now compares the identity only. Signing out, switching
  accounts, an opaque token becoming a JWT, and a change to any non-volatile
  claim still reset. Routine rotation still does not, and calling
  `resetAuthQueries()` directly is unchanged.
