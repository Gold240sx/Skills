# WorkOSAuthKitSwift — file map & notes

## Package root

`WorkOSAuthKitSwift/` — typically at  
`/Users/michaelmartell/Documents/CODE/Swift/Projects/2026/DevSpace/WorkOSAuthKitSwift`  
(sibling to the `DevSpace` app folder in this repo layout).

## Pipes-related sources

| Path | Role |
|------|------|
| `Sources/WorkOSAuthKitSwift/Pipes/PipesClient.swift` | Actor: authorize URL, token, connected account, disconnect, **`connectProvider`** (browser + poll). |
| `Sources/WorkOSAuthKitSwift/Pipes/PipesModels.swift` | `PipesProvider`, responses, `PipesConnectionStatus`, token envelope. |
| `Sources/WorkOSAuthKitSwift/Auth/AuthStore.swift` | Owns **`pipesClient`**, **`attach`** in `init` `Task`. |
| `Sources/WorkOSAuthKitSwift/UI/WorkOSPipesGitHubMark.swift` | GitHub + Pipes mark for lists/settings. |

## DevSpace app (consumer) touchpoints

| Path | Role |
|------|------|
| `DevSpaceDesktop/App/DevSpaceAuthSessionController.swift` | **`configurePipes(authStore.pipesClient)`** after auth. |
| `DevSpaceDesktop/Services/NetworkManager.swift` | GitHub token fallback: **`getAccessToken(.github)`**. |
| `DevSpaceDesktop/Services/WorkOSPipesErrorUtilities.swift` | **Already installed** / benign 400 handling. |
| `DevSpaceDesktop/Views/Pages/Integrations/GitHubIntegration.swift` | Settings-style connect UX. |
| `DevSpaceDesktop/Views/Onboarding/Steps/Group3/OnboardingStepGroup3ConnectGitHub.swift` | Onboarding connect flow. |
| `DevSpaceDesktop/App/DevSpaceAuthConfiguration.swift` | **`hasWorkOSAPIKeyForPipes`**, RBAC URL flags. |

## WorkOS docs

Use [WorkOS Docs](https://workos.com/docs) for current REST shapes; keep Swift `Codable` models in sync with **`PipesModels.swift`** and **`PipesClient`** decode paths.

## If you add a wrapper package

If “WorkOSWiftData” (or similar) wraps AuthKit: add a short section here with **types**, **entry points**, and **how it forwards to `PipesClient`**, so the agent prefers your API without guessing.
