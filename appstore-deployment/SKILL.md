---
name: appstore-deployment
description: App Store deployment compliance agent for iOS/macOS apps. Audits submissions for rejection risk, guides pre-submission preparation, and produces App Review notes, demo account packages, and appeal templates. Use when submitting to the App Store, debugging a rejection, preparing review notes, checking IAP/subscription compliance, auditing privacy manifests, reviewing permissions, evaluating AI features, or handling external purchase links.
---

# App Store Deployment Agent Skill

Reduces App Store rejection risk by applying Apple's current review criteria (guidelines last updated **February 6, 2026**) to your submission before it reaches reviewers.

> **Authority hierarchy**: Apple's App Review Guidelines → Apple Support/Developer Docs → Apple WWDC transcripts → Community evidence (Reddit/StackOverflow/blogs).  
> When sources conflict, defer to the most recent Apple document.

## Quick Approach

When invoked, run these gates in order — stop at the first blocker and fix it before continuing:

1. **Gate A – Hard submission blockers** (ITMS errors, missing manifests, broken URLs)
2. **Gate B – App completeness** (crashes, placeholders, missing demo access, broken IAPs)
3. **Gate C – Privacy & permissions** (policy, purpose strings, ATT, account deletion)
4. **Gate D – Payments & monetization** (IAP rules, storefront scope, subscription UI)
5. **Gate E – Design & metadata** (4.2 minimum functionality, hidden features, spam)
6. **Gate F – Code execution & supply chain** (private APIs, hot updates, SDK manifests)
7. **Gate G – AI, UGC, and age gates** (disclosure, moderation, declared/verified age limits)
8. **Gate H – Trust & anti-scam checks** (dark patterns, misleading pricing, manipulative flows)
9. **Gate I – Review operations & evidence** (reviewer-only bug logging, Resolution Center packets)

Then produce: **Submission readiness verdict → Reviewer path script → App Review Notes draft → escalation options**.

---

## Rejection Frequency (Apple 2024 — ~24.9% rejection rate)

Apple reviewed **7,771,599** submissions and rejected **~1,931,400** in 2024. A single rejection can cite multiple sections. Ranked by section prevalence:

| Rank | Section | Apple Anchor | Key Sublaws |
|------|---------|-------------|-------------|
| 1 | **Performance** | §2.1 App Completeness | Crashes, placeholders, missing IAP, broken links |
| 2 | **Legal** | §5.1 Privacy | Policy missing, bad purpose strings, ATT, account deletion |
| 3 | **Design** | §4.2 / §4.3 | Minimum functionality, spam, copycats |
| 4 | **Business** | §3.1 Payments | IAP required, external links, subscription UI |
| 5 | **Safety** | §1.2 UGC | Missing moderation, report/block |
| — | **Technical** | §2.5.1/2.5.2 | Private APIs, code download, hot updates |

> Apple states: **"Over 40% of unresolved issues are related to Guideline 2.1: App Completeness."**  
> Source: [developer.apple.com/app-store/review](https://developer.apple.com/app-store/review/)

---

## Gate A — Hard Submission Blockers

These prevent even entering review. Fix before submitting.

**Code signing & entitlements**
- Entitlements in the binary must match provisioning profile and App ID capabilities exactly.
- Common errors: `ITMS-90161` (invalid provisioning profile), `ITMS-9000` (invalid binary), entitlements mismatch.
- Fix: Regenerate profiles after adding/removing capabilities. Debug: `codesign -d --entitlements :- YourApp.app`.

**Launch screen storyboard (required since April 2020)**
- All iOS apps must include a LaunchScreen storyboard — apps without one are rejected.
- Xcode 16 bug: manually type the filename; verify `UILaunchStoryboardName` in Info.plist.

**Screenshot dimensions**
- Exact pixel dimensions required (1px off fails upload). Minimum: 6.7" (1290x2796) iPhone; 12.9" (2048x2732) iPad.
- Screenshots must reflect current build UI — outdated screenshots trigger §2.3.

**App size limits**
- iOS/visionOS: 4 GB max, __TEXT: 500 MB. watchOS: 75 MB hard limit.
- Cellular download: 200 MB default threshold — above requires Wi-Fi.
- Use Background Assets / On Demand Resources for large content.

**App Transport Security (ATS)**
- Global `NSAllowsArbitraryLoads = YES` requires documented justification and may be rejected.
- Use domain-specific `NSExceptionDomains` instead. iOS 17+ has stricter IP-based restrictions.

**Privacy manifests — two separate obligations (both can block upload)**

1. **Third-party SDKs on Apple's list** — When you add or update an app that includes a listed binary SDK, that SDK must ship a valid `PrivacyInfo.xcprivacy` and (for binary deps) a valid signature. Common ITMS error: `ITMS-91061: Missing privacy manifest`.  
   - List (updates over time): [developer.apple.com/support/third-party-SDK-requirements](https://developer.apple.com/support/third-party-SDK-requirements/)  
   - Fix: Upgrade the SDK; do **not** copy the vendor's data-collection story into your app's manifest as a substitute for their missing file (Apple expects the SDK bundle to carry its own manifest).

2. **Required reason APIs in *your* code** — If *your* app or any embedded executable uses Apple's "required reason" API categories (file timestamp, disk space, UserDefaults, system boot time, active keyboard, etc.), **your** app (or that executable's bundle) must declare approved reasons in `PrivacyInfo.xcprivacy` under `NSPrivacyAccessedAPITypes`. Starting **May 1, 2024**, apps without these declarations are not accepted. **Fingerprinting is never allowed**, regardless of user consent.  
   - Reference: [Describing use of required reason API](https://developer.apple.com/documentation/bundleresources/privacy_manifest_files/describing_use_of_required_reason_api)  
   - Xcode: **File → Project → Generate Privacy Report** aggregates SDK + app manifests.

**Broken required URLs**
- Support URL and Privacy Policy URL must return HTTP 200 at time of submission.
- Apple calls these out explicitly as common "unresolved issues."
- Fix: Run a URL health check script before every submission.

**Export Compliance**
- If the app uses encryption (even HTTPS), set `ITSAppUsesNonExemptEncryption` in Info.plist appropriately.
- Without this, Apple will prompt during upload and block distribution until answered.
- Reference: [developer.apple.com/documentation/security/complying_with_encryption_export_regulations](https://developer.apple.com/documentation/security/complying_with_encryption_export_regulations)

**Distribution scope & compliance (outside guideline text but blocks release)**
- **App Store** — Full [App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/). Prep: [App Review](https://developer.apple.com/app-store/review/).
- **EU alternative marketplaces / Web Distribution / Japan marketplace** — Subset of guidelines + Notarization; use **Highlight Notarization Review Guidelines Only** in the guidelines UI. Still scanned for malware/safety. [Notarization (ASC)](https://developer.apple.com/help/app-store-connect/managing-alternative-distribution/submit-for-notarization), [DMA & EU apps](https://developer.apple.com/support/dma-and-apps-in-the-eu/), [Web Distribution EU](https://developer.apple.com/support/web-distribution-eu/).
- **Mac App Store** — Extra rules in §2.4.5 (sandbox, no third-party installers, MAS-only updates). See [REFERENCE.md — Mac App Store](REFERENCE.md#mac-app-store-additional-rules-245).
- **EU DSA trader** — Traders distributing in the EU must complete verified contact display in App Store Connect (can block submission flow). [Manage EU DSA trader requirements](https://developer.apple.com/help/app-store-connect/manage-compliance-information/manage-european-union-digital-services-act-trader-requirements/).

---

## Gate B — App Completeness (§2.1)

The single largest rejection driver. Apple requires submissions be **"final versions."**

**Mandatory checklist:**
- [ ] App tested on physical device (not only Simulator) via TestFlight build
- [ ] Zero crashes on cold start, onboarding, login, core flows, and checkout
- [ ] No placeholder text ("Lorem ipsum"), empty screens, or stub features
- [ ] All in-app purchase products submitted with the binary and visible to reviewer
- [ ] Demo account provided (or Apple-approved demo mode): username, password, no 2FA blocking access, region matching reviewer environment
- [ ] Backend services live and accessible from outside your VPN/network
- [ ] Support URL and Privacy Policy URL are functional
- [ ] Notes for Review explain every non-obvious feature, IAP purchase path, and any hardware requirement

**IAP-specific "catch-22" loop (very common):**  
Products showing blank/missing pricing during review → rejected under 2.1(b). Fix: Submit IAPs together with the binary in App Store Connect. Ensure products are approved (not just created) before submitting the build.

**Receipt validation pattern that satisfies reviewers:**  
`/verifyReceipt` production first → if status 21007, fallback to sandbox. Apple's own rejection message text confirms this sequence.

**Reference:** [developer.apple.com/app-store/review/guidelines/#app-completeness](https://developer.apple.com/app-store/review/guidelines/#app-completeness)  
**Community case:** RevenueCat ultimate guide — [revenuecat.com/blog/growth/the-ultimate-guide-to-app-store-rejections](https://www.revenuecat.com/blog/growth/the-ultimate-guide-to-app-store-rejections/)

---

## Gate C — Privacy & Permissions (§5.1)

Second-largest rejection bucket. Three artifacts must **agree**: privacy policy content, App Privacy nutrition label, and runtime behavior.

**Purpose strings — literal wording matters:**  
Bad: `"We need your location."`  
Good: `"Your location is used to show nearby coffee shops on the map."`  
Rejection message pattern: *"Your app requests access to [resource] but does not clearly explain why."*

**Account deletion — required since June 2022:**
- Must be in-app initiated (not email-only or support-ticket-only)
- Must delete account AND associated data (unless legally required to retain)
- Must be easy to find (Settings → [Account] → Delete Account)
- Reference: [developer.apple.com/support/offering-account-deletion-in-your-app](https://developer.apple.com/support/offering-account-deletion-in-your-app/)

**Mandatory login gating rule:**
- If login is required, document in Notes for Review *why* the feature requires identity (cross-device state, progression, personalization)
- Allow guest/browse mode for features that don't need an account
- Rejection pattern: *"Your app requires users to log in before they can access features that are not account-based."*

**ATT (App Tracking Transparency):**
- Use `AppTrackingTransparency` framework for any cross-app/cross-site tracking
- Cannot gate features on user granting tracking permission
- Reference: [developer.apple.com/documentation/apptrackingtransparency](https://developer.apple.com/documentation/apptrackingtransparency)

**Third-party AI disclosure — explicit Apple requirement (§5.1.2(i)):**  
> *"You must clearly disclose where personal data will be shared with third parties, including with third-party AI, and obtain explicit permission before doing so."*

**Reference:** [developer.apple.com/app-store/review/guidelines/#privacy](https://developer.apple.com/app-store/review/guidelines/#privacy)

---

## Gate D — Payments & Monetization (§3.1)

**Core IAP rule:** Unlocking features, subscriptions, in-game currency, premium content → **must use StoreKit IAP.** No QR codes, license keys, AR markers, or direct Stripe links (outside allowed exceptions).

**External purchase links — storefront-dependent (frequently misunderstood):**

| Storefront | External CTA allowed? | Entitlement needed? |
|-----------|----------------------|-------------------|
| United States | Yes | No (as of current guidelines) |
| EU (iOS/iPadOS) | With StoreKit External Purchase Link Entitlement | Yes |
| Netherlands dating apps | With specific entitlement | Yes |
| South Korea | With StoreKit External Purchase Entitlement | Yes |
| All others | No | N/A |

**Subscription paywall — minimum required disclosures:**
- Subscription price and billing period must be **prominent and legible**
- The **amount actually billed** must be the most prominent pricing element in the layout
- Auto-renewal must be disclosed
- "Cancel anytime" text cannot be the only subscription info shown
- "Restore Purchases" button required
- Trial period terms must be shown before purchase
- Sign-up screen should include subscription name/duration and the service/content provided

**Rejection pattern:** App Review uses §3.1.2 as a catch-all for paywall design problems. If price/period aren't prominent enough → increase contrast and font size.

**B2B SaaS edge case:** Even if your app serves enterprise customers who purchased on the web, in-app digital access (features, content, subscriptions) still requires IAP unless the app qualifies as "multiplatform service" or "enterprise service" under §3.1.3.

**Reference:** [developer.apple.com/app-store/review/guidelines/#payments](https://developer.apple.com/app-store/review/guidelines/#payments)

---

## Gate E — Design & Metadata (§4.2, §2.3)

**Minimum Functionality (§4.2):** App must "elevate beyond a repackaged website."  
Red flags: WebView as primary experience, app equivalent to browsing a URL, no offline/native value, marketing-only content.  
Fix: Add meaningful native features — device sensors, push notifications, offline mode, native navigation, deep linking.

**Hidden features (§2.3.1):** Any feature reachable via deep link, debug gesture, or feature flag that reviewers can't discover through normal use = hidden feature = rejection.  
Fix: Document every non-obvious feature path in Notes for Review with specificity.

**Spam (§4.3):** Multiple Bundle IDs for near-identical apps (different regions/teams/languages) = removal risk.  
Fix: One app with in-app variations or in-app purchase.

**Copycats (§4.1):** Closely mirroring another app's icon, name, or UI = rejection + potential expulsion.

**App Clips / Widgets / Extensions (often overlooked):**
- App Clips cannot include advertising.
- Widgets, extensions, and notifications must be directly related to app content/functionality.
- Siri intents must match what the app actually does; avoid registering irrelevant intents.

**Reference:** [developer.apple.com/app-store/review/guidelines/#minimum-functionality](https://developer.apple.com/app-store/review/guidelines/#minimum-functionality)

---

## Gate F — Code Execution & Supply Chain (§2.5)

**No code download that changes functionality (§2.5.2):**
- Hot-update frameworks (JSPatch, Rollout, code-push patterns) that modify production JS bundles → rejected
- Rejection message: *"Your app downloads, installs, or executes code that introduces or changes features or functionality."*
- Allowed: Content/config updates that don't add new executable behavior
- Allowed narrow exception: Educational apps where code is user-viewable and user-editable
- Community fix for React Native: Remove or lock hot-update path; ensure release builds don't include dev server URLs

**Private APIs (§2.5.1):**
- `prefs:root://` and similar Settings deep links are private / unsupported → rejection risk. Prefer opening the app's settings page via `UIApplication.openSettingsURLString` (or SwiftUI `openSettingsURLString`) so the user lands in **your** app's settings, not arbitrary system panes.
- Private selectors / SPI in your code or in a dependency → automated rejection. Audit with `strings` / static analysis; update or replace SDKs that embed private symbols (IOKit private I/O, undocumented SpringBoard APIs, etc.).
- StackOverflow pattern: private IOKit or entitlement misuse flagged; fix by updating/removing the offending SDK or code path.

**Reference:** [developer.apple.com/app-store/review/guidelines/#software-requirements](https://developer.apple.com/app-store/review/guidelines/#software-requirements)

---

## Gate G — AI Features, UGC, and Age Gates (§1.2, §4.7, §5.1.2)

**If your app has AI-generated content visible to users:**
- Treat it as UGC: must have report/block/filter mechanisms (§1.2)
- Must have published contact information
- If AI can produce NSFW content, hide by default and only enable via website-side control

**If AI inference is server-side:**
- User data sent off-device → must appear in App Privacy answers and privacy policy
- If a third-party AI provider receives personal data → explicit disclosure + consent required (§5.1.2(i))

**"Coding assistant" apps / LLM code generation — §2.5.2 interpretation:**
- Generating code that the user views/edits (educational exception) = generally OK
- Downloading and executing new code that changes the app's own behavior = rejected
- Mini-apps, chatbots, plug-ins under §4.7 are allowed but require full §5.1 and §1.2 compliance

**Kids Category + AI:** Zero third-party analytics/advertising. AI features that could expose kids to inappropriate content are especially scrutinized.

**Creator apps / age-gated content:** If the app allows creation or discovery of content that can exceed the app's age rating, require a way to flag that content and restrict access using verified or declared age. Keep **App Store Connect age rating** answers current with the [2026 age rating system](https://developer.apple.com/news/upcoming-requirements/?id=07242025a) and [definitions](https://developer.apple.com/help/app-store-connect/reference/age-ratings-values-and-definitions/).

**Multi-platform targets (tvOS, watchOS, visionOS):** Privacy manifests and required-reason API rules apply per Apple docs across these platforms; validate each target's `Info.plist`, extensions, widgets, and embedded binaries separately.

---

## Gate H — Trust, Dark Patterns, and Anti-Scam Enforcement (§3.1.2, §5.6)

Apple increasingly enforces **customer trust**, not just technical compliance.

**High-risk patterns:**
- Misleading trial emphasis where the billed amount is visually subordinate
- Subscription flows that obscure renewal price, duration, or cancellation
- Forcing reviews, downloads, contact uploads, tracking permission, or other store-related actions to unlock access
- Charging for built-in OS capabilities like push notifications, camera access, or iCloud storage
- Manipulative wording, false scarcity, bait-and-switch pricing, or features promised but not delivered

**Reference:** [developer.apple.com/app-store/review/guidelines/#payments](https://developer.apple.com/app-store/review/guidelines/#payments)  
**Reference:** [developer.apple.com/app-store/review/guidelines/#legal](https://developer.apple.com/app-store/review/guidelines/#legal)

---

## Gate I — Review Operations & Evidence

Use this when the issue only reproduces in App Review but not internally.

**Reviewer-only issue playbook (high success rate):**
- Add temporary structured diagnostics for login, paywall load, and API error paths.
- Log non-PII correlation data: build number, timestamp, endpoint, status code, device model/OS if available.
- Correlate logs with App Review rejection timestamp and attach findings in Resolution Center.
- Include screenshot/video and exact repro path in your response packet.

**Why this matters:** a frequent rejection loop is \"can't log in / can't proceed\" that cannot be reproduced by internal testers. Evidence-backed replies reduce back-and-forth and unblock review faster.

**Operational reminders from Apple guidance:**
- Repeated rejections for the same issue can slow review.
- Metadata-only rejections can often be fixed and resubmitted without a new binary.

---

## Reviewer Path Script Template

Produce this for every submission. Fill in all `[brackets]`.

```
App Review Notes

App concept: [1-2 sentence description of what the app does and for whom]

Demo account:
  Username: [email or username]
  Password: [password]
  Region/storefront: [e.g. United States]
  Notes: [any pre-seeded data the reviewer needs to see, no 2FA active]

Key feature walkthrough:
  1. [Step to reach Feature A — required for reviewer to find it]
  2. [Step to trigger in-app purchase paywall — exact tap path]
  3. [Step to find account deletion — Settings → Profile → Delete Account]
  4. [Step to access AI feature, if present]

Backend: Live and accessible. No VPN required.

In-app purchases: [List product IDs submitted with this binary]

Non-obvious behaviors: [Anything that might look like a bug but is intentional]

Attachment: [demo video URL or note if hardware is required]
```

---

## After Rejection: Resolution Loop

```
Rejected → Read guideline clause cited exactly
         → Reproduce on a TestFlight build on real device
         → Classify: metadata-only fix OR requires new binary?
         → Metadata only: fix + reply in Resolution Center (no new binary needed)
         → New binary: fix code → QA → upload new build → reply with notes
         → Disagree: submit ONE appeal per rejected submission via App Store Connect
         → Appeal upheld: resolved
         → Appeal denied: fix per guidelines and resubmit
```

**Appeal tips (from Apple's own guidance):**
- Cite the exact guideline clause and explain compliance specifically
- Do not submit multiple appeals for the same rejection
- Bug-fix-only submissions for live apps can bypass some violations (except legal/safety)
- You can attach screenshots, demo videos, and supporting documents until you resubmit
- If only part of a multi-item submission is rejected, you can remove rejected items and continue with accepted ones
- Apple offers App Review appointments for pre-review/process guidance on difficult submissions

**Reference links:**  
- Appeals: [developer.apple.com/contact/app-store/?topic=appeal](https://developer.apple.com/contact/app-store/?topic=appeal)
- Expedited review: [developer.apple.com/contact/app-store/?topic=expedite](https://developer.apple.com/contact/app-store/?topic=expedite)

---

## Reference Files

- **[REFERENCE.md](REFERENCE.md)** — Full rejection database with community case studies, Apple doc anchors, Reddit/StackOverflow patterns, fix code examples
- **[EDGE-CASES.md](EDGE-CASES.md)** — AI policy, hot updates, privacy manifests, external payments, UGC edge cases, regulated domains
- **[CHECKLIST.md](CHECKLIST.md)** — Printable pre-submission checklist and reviewer kit templates
- **[ADDENDUM.md](ADDENDUM.md)** — Vibe coding crackdown (March 2026), age rating overhaul, visionOS requirements, app size limits, ITMS error codes, ATS exceptions, launch screen, code signing, TestFlight vs App Store review, inactive app removal, screenshot requirements, local network permission, notification timing, post-approval quality (§5.6.4)

**Maintenance:** Rejection statistics (e.g. 2024 transparency counts) should be refreshed when Apple publishes new annual reports; gate logic and guideline links stay authoritative until Apple updates the guidelines page.
