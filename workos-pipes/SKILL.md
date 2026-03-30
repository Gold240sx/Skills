---
name: workos-pipes
description: >-
  WorkOS Pipes via the WorkOSAuthKitSwift Swift package: AuthStore.pipesClient,
  OAuth connect flows, tokens, and backend proxy routes. Use when wiring or
  debugging GitHub (or other providers) through Pipes, WorkOSAuthKitSwift,
  AuthStore, PipesClient, DevSpace integrations, or data-integrations APIs.
---

# WorkOS Pipes + **WorkOSAuthKitSwift**

## Canonical package (this monorepo)

All Pipes client logic for DevSpace lives in the local Swift package:

**`WorkOSAuthKitSwift/`**  
(full path example: `.../DevSpace/WorkOSAuthKitSwift`)

- SwiftPM target: `WorkOSAuthKitSwift`
- Pipes implementation: `Sources/WorkOSAuthKitSwift/Pipes/`
- **Do not assume a separate “WorkOSWiftData” package** for Pipes in this repo; if another package wraps AuthKit, document it in that repo’s skill or extend this skill’s `reference.md`.

## How Pipes is wired in WorkOSAuthKitSwift

1. **`AuthStore`** (`Auth/AuthStore.swift`) creates a single **`PipesClient(configuration:)`** and exposes it as **`authStore.pipesClient`**.
2. On init, `AuthStore` runs `await pipesClient.attach(authStore: self)` so the actor can read `userInfo.sub`, org id, and **`validAccessToken()`** for **proxy** calls.
3. Apps **do not** manually construct `PipesClient` unless testing; use **`environment(\.authStore)`** / injected `AuthStore` and **`store.pipesClient`**.

## Transport (same as WorkOS docs)

| Mode | `WorkOSConfiguration` | Pipes HTTP auth |
|------|-------------------------|-----------------|
| **Proxy (production)** | `backendUrl` set | `Bearer <user access token>` |
| **Direct (dev only)** | `workosApiKey` (`sk_…`) | `Bearer sk_…` — **never ship in App Store / public clients** |

`PipesClient` picks proxy if no API key but `backendUrl` is set; otherwise direct. Misconfiguration throws `AuthError.configurationError` with a message pointing at RBAC URL vs API key.

## Backend proxy ↔ WorkOS (implement on your API)

| Client → your API | WorkOS |
|-------------------|--------|
| `POST /pipes/:slug/authorize` | `POST /data-integrations/:slug/authorize` |
| `POST /pipes/:slug/token` | `POST /data-integrations/:slug/token` |
| `GET /pipes/:slug/connected-account` | `GET /user_management/users/:id/connected_accounts/:slug` |
| `DELETE /pipes/:slug/connected-account` | `DELETE` … same |

Slug examples: `github`, `slack`, `google`, `salesforce` (`PipesProvider`).

## API surface in the package (`PipesClient.swift`)

- **`getAuthorizationURL(provider:returnTo:)`** — proxy: optional `return_to`; direct: sends `user_id`, optional `organization_id`.
- **`connectProvider(provider:pollingInterval:timeout:)`** @ MainActor — **native flow that worked in DevSpace**: open WorkOS URL in **system browser**, **poll** `getConnectionStatus` until connected (no custom-scheme `return_to`; see comments in `PipesClient`).
- **`getConnectionStatus` / `getConnectedAccount`** — `404` → disconnected / `nil`.
- **`getAccessToken(provider:)`** — returns `PipesAccessTokenEnvelope`: check **`active`**, then **`accessToken?.token`**; read **`error`** (`needs_reauthorization`, `not_installed`, …).
- **`disconnect(provider:)`** — `connectProvider` pre-clears stale **`needs_reauthorization`** via disconnect when needed.

Types: **`PipesModels.swift`** (`PipesProvider`, `PipesConnectionStatus`, envelopes).

## DevSpace integration (verified patterns)

Wire Pipes into networking after auth is ready:

- **`DevSpaceAuthSessionController`** → **`NetworkManager.shared.configurePipes(authStore.pipesClient)`** so GitHub REST calls can fall back to a Pipes token when the user has no manual PAT (`NetworkManager.swift`).

UI / flows:

- **`GitHubIntegration.swift`** — connect / disconnect / status via **`store.pipesClient`**, gated on **`DevSpaceAuthConfiguration.hasRBACService || hasWorkOSAPIKeyForPipes`**.
- **`OnboardingStepGroup3ConnectGitHub.swift`** — parallel onboarding GitHub row; **`connectProvider(.github)`** and refresh status with **`getConnectionStatus`**.
- **`WorkOSPipesErrorUtilities`** — treat WorkOS **“already installed” / “already connected”** style **400** as *effectively connected*, not a hard failure (matches real GitHub App + Pipes behavior).

Branding helper: **`WorkOSPipesGitHubMark.swift`** in the package.

## Checklist for new Pipes usage in an AuthKit app

1. Configure **`WorkOSConfiguration`** (`backendUrl` or dev `workosApiKey`).
2. Use existing **`AuthStore`**; call **`await authStore.pipesClient.connectProvider(.github)`** (or other provider) from **MainActor** UI.
3. For API calls, **`getAccessToken`** and handle **`!active`** / **`error`**.
4. Map “benign” errors with **`WorkOSPipesErrorUtilities`** where appropriate (DevSpace copy or equivalent).

## Pitfalls (from production debugging)

- **Stacking `GlobalModalManager.present`** on top of content that is *already* the global modal **replaces** the first modal and **destroys** that subtree — `goToNext()` and similar state then run off-screen. Use an **in-flow overlay** or a single modal owner (see DevSpace post-sign-in onboarding subscription overlay fix).
- **Custom URL `return_to`** — native AuthKit **`connectProvider`** omits it and **polls**; don’t assume scheme callbacks for Pipes completion.
- **Poll loop** — do not treat **`needs_reauthorization`** as immediate failure while polling; user may still be in the browser.
- **`sk_` in shipped apps** — forbidden; use RBAC/proxy only for release.

## More detail

- [reference.md](reference.md) — file map under `WorkOSAuthKitSwift` + WorkOS doc pointers.
