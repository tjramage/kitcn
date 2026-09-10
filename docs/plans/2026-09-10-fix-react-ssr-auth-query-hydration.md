# Fix React SSR auth query hydration

Objective:
Stop `CRPCProvider` clearing authenticated cRPC queries when Convex confirms a token the client already held; done when a server-rendered authenticated query survives hydration and the React suite passes; plan docs/plans/2026-09-10-fix-react-ssr-auth-query-hydration.md.

Goal plan:
docs/plans/2026-09-10-fix-react-ssr-auth-query-hydration.md

Template:
docs/plans/templates/task.md

Task source:
- type: single-PR task
- id / link: PR pending, fill in after opening
- title: authenticated SSR cRPC hydration is cleared when Convex confirms the same token

Task PR:
pending, fill in after opening

## Problem

`CRPCProviderInner` resets every auth-bound query when either the token identity
or `isAuthenticated` changes. The flag reports whether Convex has accepted the
current token, not who holds it, so it turns from false to true on every
server-rendered page once Convex validates the token that arrived in the HTML.
The reset then erases the queries the same render hydrated, and the screen loses
rows it had already shown.

## Canonical reproduction path

`example/src/components/providers.tsx` prefetches
`crpc.user.getCurrentUser.queryOptions(undefined, { skipUnauth: true })` and
wraps the tree in `HydrateClient`, and
`example/src/lib/convex/convex-provider.tsx` passes that request's token to
`ConvexAuthProvider` and mounts `CRPCProvider` under it. Any client reading that
query loses the hydrated result on load.

## Fix

`packages/kitcn/src/react/context.tsx` compares the token-derived identity only.
`resolveAuthIdentity` already collapses a JWT to its non-volatile claims, so
routine rotation stays a no-op while signing out, switching accounts, an opaque
token becoming a JWT, and a change to any other claim all produce a different
identity. The unused `isAuthenticated` read, ref field and effect dependency are
gone, and the ref is now `previousIdentityRef`. Explicit
`resetAuthQueries()` calls are untouched.

## Tests

`packages/kitcn/src/react/context.test.tsx` gains one case: a JWT present on the
first render, the flag moving false to true, back to false, and to true again
with no reset, then the token going null and resetting once. The existing
account-switch, claim-change, opaque-to-JWT and rotation cases stay as they are.

## Verification

- `bun test packages/kitcn/src/react/`
- `bun lint`
- `bun run lint:fix`

`bun run build:pkg` and `bun typecheck` fail on `main` at 5b0bd5a1 for unrelated
reasons: the package build stops on `better-call is located in node_modules but
is not included in inlineOnly`, and the `test-convex` typecheck cannot resolve
`kitcn/auth/*` without that build. Both reproduce on a pristine checkout of the
same commit.
