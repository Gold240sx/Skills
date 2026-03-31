---
name: appstore-deployment
description: App Store deployment compliance agent for iOS/macOS apps. Audits submissions for rejection risk, guides pre-submission preparation, and produces App Review notes, demo account packages, and appeal templates. Use when submitting to the App Store, debugging a rejection, preparing review notes, checking IAP/subscription compliance, auditing privacy manifests, reviewing permissions, location and advertising/ATT rules, Apple or third-party trademarks and marketing assets, encryption export compliance, safety or objectionable content (violence, weapons, gambling), speech/UGC policy, sandboxing and system-level access, evaluating AI features, handling external purchase links, or aligning with Apple Developer Program License Agreement (ADPLA) program requirements.
---

# App Store Deployment Agent Skill

Reduces App Store rejection risk by applying Apple's current review criteria (guidelines last updated **February 6, 2026**) to your submission before it reaches reviewers.

> **Authority hierarchy**: Apple's **App Review Guidelines** → **Program Requirements** (including Human Interface Guidelines and API docs incorporated into the [Apple Developer Program License Agreement](https://developer.apple.com/support/terms/apple-developer-program-license-agreement/) — “ADPLA”) → Apple Support/Developer Docs → Apple WWDC transcripts → Community evidence (Reddit/StackOverflow/blogs).  
> When sources conflict, defer to the most recent Apple document. **Guidelines and ADPLA overlap but are not identical**: review can cite either; binary behavior must satisfy documented APIs, privacy rules, and contractual program requirements, not only guideline prose.

## Quick Approach

When invoked, run these gates in order — stop at the first blocker and fix it before continuing:

1. **Gate A – Hard submission blockers** (ITMS errors, missing manifests, broken URLs, **encryption/export**)
2. **Gate B – App completeness** (crashes, placeholders, missing demo access, broken IAPs)
3. **Gate C – Privacy & permissions** (policy, purpose strings, **location & ads**, ATT, account deletion)
4. **Gate D – Payments & monetization** (IAP rules, storefront scope, subscription UI)
5. **Gate E – Design & metadata** (4.2 minimum functionality, hidden features, spam, **IP & marks**)
6. **Gate F – Code execution & supply chain** (private APIs, hot updates, SDK manifests)
7. **Gate G – AI, UGC, and age gates** (disclosure, moderation, declared/verified age limits)
8. **Gate K – Safety, speech, gambling, sandbox & system access** (§1.1 violence/weapons, §5.3, containers vs privileged APIs)
9. **Gate H – Trust & anti-scam checks** (dark patterns, misleading pricing, manipulative flows)
10. **Gate I – Review operations & evidence** (reviewer-only bug logging, Resolution Center packets)
11. **Gate J – ADPLA & program requirements** (submission duties, TestFlight contract rules, restricted APIs/data uses called out in the license)

Then produce: **Submission readiness verdict → Reviewer path script → App Review Notes draft → escalation options**.

---

## Gate J — ADPLA & program requirements (contract + §3.3)

Use this gate when auditing **contractual** obligations and **Program Requirements** embedded in the ADPLA. These often align with App Review but can bite in legal, enterprise, or niche API cases.

**Submission & honesty (Agreement §6.1, §3.2(f))**
- Do **not** hide, misrepresent, or obscure features from review; you must cooperate with reasonable information requests.
- If the app **connects to a physical device** (including MFi accessories), **inform Apple in App Store Connect**: disclose the means of connection (e.g. BLE, Lightning, headphone jack, other protocol) and identify **at least one** physical device the app is designed to work with. Apple may request access or samples **at your expense** (samples may not be returned).
- Any post-submission change to functionality (including IAP-delivered functionality) generally requires **resubmission** for App Store / Custom App Distribution.

**Service providers (§2.9)**
- A third-party **may not** submit your app to the App Store or operate **TestFlight** on your behalf. (They may still host servers, etc., under a written agreement at least as protective of Apple as the ADPLA.)

**Certificates & signing (§5)**
- Safeguard distribution and submission certificates and keys; **do not** put App Store distribution certificates in shared cloud repos for third-party use. Notify Apple promptly if you believe keys are compromised (`product-security@apple.com`).

**Multi-platform visibility (§6.3, §6.4)**
- iOS/iPadOS apps submitted to the App Store may also be offered on **macOS** and **visionOS** via the App Store unless you **opt out** in App Store Connect — confirm rights, compatibility, and testing, or opt out.
- **CarPlay**: iOS app widgets may surface on CarPlay; use **`disfavorLocations`** when a widget is unsuitable for driving (e.g. heavy interaction, requires unlock).

**TestFlight — external testers (§6.5, §7.4)**
- **External** TestFlight distribution requires an **Apple-reviewed** build first. Updates with **significant changes** need re-review per App Store Connect process.
- **No fees** to participate in TestFlight; digital purchases in TestFlight builds must be **free** and for beta only.
- Prohibited: using TestFlight to **circumvent** the App Store, run perpetual demos, or **solicit App Store ratings**.
- **Kids-focused apps**: verify external beta testers meet **age of majority** in their jurisdiction.

**Operations & analytics (§6.7–6.8)**
- Data from Apple’s **developer analytics** is for **improving your apps** only — no handoff to third parties except a **Service Provider** under strict use limits; no cross-developer aggregation.
- Apps in the store must stay **compatible with the currently shipping OS**; Apple may remove apps that fall behind.

**Privacy & data — program text beyond typical guideline bullets**
- **Recordings** (camera, mic, screen, etc.): show a **clear, conspicuous** in-app indicator while recording; do not design for **secret** recording of others.
- **Sensitive Content Analysis**: do **not** send **off device** any signal about whether content was flagged as containing nudity.
- **Face Data**: use only for a **direct app purpose**; no advertising, authentication, or marketing; no selling to brokers; transfer off-device only with **clear consent** and for the stated function.
- **Health / HIPAA**: the ADPLA restricts using Apple Software/Services (including iCloud / CloudKit in scope of the terms) to create, store, or transmit **PHI** in ways that would make Apple your **business associate** — treat regulated health stacks as **legal + architecture** review, not checklist-only.
- **Declared Age Range / Significant App Update APIs**: use only for **age-appropriate features** or **legal** requirements; no selling that data to data brokers.
- **IDFA / AdSupport**: use only for **serving advertising**; honor **Tracking Preference**; after user resets the advertising identifier, do **not** link old and new IDs or derived data.

**In-App Purchase — Attachment 2 hooks**
- **Keyboard extensions** must **not** offer IAP **inside** the keyboard UI (IAP OK elsewhere in the host app).
- **US App Store storefront**: for **one-time** digital content IAP, before purchase you must clearly state the user is buying a **license**, with a link to full terms including **Apple Media Services Terms and Conditions** (and your EULA if applicable) — see Attachment 2 §3.2.

**Maps & Weather — required notice text (§3.3.3(F), WeatherKit Attachment 8)**
- **Real-time route / navigation** apps: EULA must include Apple’s required **“YOUR USE OF THIS REAL TIME ROUTE GUIDANCE APPLICATION IS AT YOUR SOLE RISK. LOCATION DATA MAY NOT BE ACCURATE.”** (exact wording per current ADPLA / docs).
- **Real-time weather guidance** via WeatherKit: EULA must include the **WeatherKit** disclaimer block required in Attachment 8 (accuracy / sole risk).

**Foundation Models (§3.3.8(I))**
- On-device / Apple Intelligence **Foundation Models** use is subject to Apple’s **[Foundation Models Framework Acceptable Use Requirements](https://developer.apple.com/apple-intelligence/acceptable-use-requirements-for-the-foundation-models-framework)**; keep **Adapters** compatible with the shipping model or risk OS incompatibility (§6.8).

**Regional addenda (verify current Apple Materials)**
- **Japan** — Alternative App Marketplaces, **Alternative Payment Processing**, and **Out-of-App Offers** have detailed design, entitlement, browser, prominence, age-gating, and reporting rules (**Attachment 12**). See [ADDENDUM.md — Japan ADPLA commerce](ADDENDUM.md#japan-app-store-adpla-attachment-12-commerce--distribution).
- **China mainland storefront** — **Attachment 13** amends **paid** commission rates (e.g. **25%** standard / **12%** small business and qualifying cases effective **March 15, 2026** per ADPLA text); confirm current Paid Applications Agreement for accounting.

**Notarization (macOS §5.3)**
- You may **not** tell users that Apple “approved” or “secured” your app because it was notarized; notarization is **not** a malware or safety guarantee from Apple’s perspective.

**EU — DSA & P2B (Schedule 1 Exhibit D)**
- Developers established in the EU can use Apple’s **DSA redress** information for account/app removal disputes: [apple.com/legal/dsa/redress-options](https://www.apple.com/legal/dsa/redress-options). **P2B** (platform-to-business) complaints for eligible regions: [developer.apple.com/contact/p2b](https://developer.apple.com/contact/p2b/).

**Illegal content & platform safety (§3.2(b))**  
- The ADPLA prohibits using Apple tools or services for unlawful activity or content — including material that **facilitates child sexual exploitation or abuse**. Such content is also a **hard rejection** under **Safety** guidelines (**§1.1**). Treat as **legal + safety + immediate removal** if ever implicated.

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

**Export compliance & encryption (upload + ADPLA)**
- If the app uses **encryption** (TLS, crypto libraries, proprietary crypto, etc.), set **`ITSAppUsesNonExemptEncryption`** in `Info.plist` accurately and complete the **encryption / export** questionnaire in App Store Connect. Without this, upload or distribution can stall until answered.
- **Scope is broad**: many apps answer “yes” because they use HTTPS or standard OS crypto; follow Apple’s wizard and docs for **exemptions** (e.g. only exempt categories).
- **Stronger cases**: custom crypto, VPN, secure messaging, or regulated markets may require **export classification** work (e.g. **EAR** / **BIS** self-classification or **CCATS** where applicable) and accurate **Schedule 1**–style certifications when you deliver binaries to Apple. Treat this as **legal + security**, not only a plist checkbox.
- **macOS notarization upload (§5.3)**: ADPLA prohibits uploading apps subject to certain **export-controlled** categories without authorization; align notarization payloads with your export story.
- Reference: [Complying with encryption export regulations](https://developer.apple.com/documentation/security/complying_with_encryption_export_regulations) · [App Store Connect — export compliance](https://developer.apple.com/help/app-store-connect/reference/app-information/export-compliance-documentation-for-encryption)

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

**Third-party AI disclosure — explicit Apple requirement (§5.1.2(i)):**  
> *"You must clearly disclose where personal data will be shared with third parties, including with third-party AI, and obtain explicit permission before doing so."*

**Location services (§5.1.5 + ADPLA §3.3.3(F))**  
- Use **Core Location / MapKit / location-based** features only when **directly relevant** to what the app does.  
- **Do not** market or design for **autonomous vehicle control**, **emergency / life-saving** reliance, or other disallowed positioning (see guideline text and ADPLA for navigation/weather disclaimer requirements in **Gate J**).  
- **Notify and obtain consent** before collecting, transmitting, or using location; **purpose strings** and in-app explanations must match real use (**§5.1**).  
- **Do not** disable, spoof, or wrap Apple’s **system consent UI** for location (or contacts, photos, calendar, etc.) — accurate copy in permission dialogs only (**ADPLA §3.3.3(F)(iv)**).  
- **“Always” location**: provide a clear **user benefit** when requesting continuous access.  
- **Custom map overlays**: you are responsible for alignment and correctness when combining your data with **Apple Maps**.  
- Reference: [§5.1 Legal — Location Services](https://developer.apple.com/app-store/review/guidelines/#legal) · [HIG — accessing private data](https://developer.apple.com/design/human-interface-guidelines/patterns/accessing-private-data/)

**Advertising, tracking & identifiers**  
- **ATT** (`AppTrackingTransparency`): required for **cross-app / cross-site tracking** per Apple’s rules; cannot **gate core features** on tracking consent.  
- **IDFA / AdSupport (ADPLA §3.3.3(E))**: use **only** to **serve advertising**; check **Tracking Preference** before using IDFA for ads; after user **resets** the advertising identifier, do **not** link old and new IDs or derived data.  
- **Ad Network APIs** (if granted): use **only** for **ad validation / conversion** per Apple’s terms — no combining validation payloads with other user profiles.  
- **Kids Category**: **no** third-party analytics or third-party advertising in apps **primarily for kids** (**§1.3**); stricter rules for minors’ data.  
- **APNs / local notifications (Attachment 1)**: no **spam** or **phishing**; **promotional** push only with **explicit in-app opt-in** language and in-app **opt-out**, except Pass-related promotions tied to the Pass. Do not use notifications to **evade** privacy or tracking rules.  
- **Network Extension / “Access Wi‑Fi Info” (ADPLA §3.3.3(G))**: entitlements are for **networking functionality**, **not** ads, profiling, or bypassing user privacy settings (e.g. inferring location when location is off).  
- Reference: [§5.1 Privacy](https://developer.apple.com/app-store/review/guidelines/#privacy) · [ATT](https://developer.apple.com/documentation/apptrackingtransparency)

**Reference:** [developer.apple.com/app-store/review/guidelines/#privacy](https://developer.apple.com/app-store/review/guidelines/#privacy)

---

## Gate D — Payments & Monetization (§3.1)

**Core IAP rule:** Unlocking features, subscriptions, in-game currency, premium content → **must use StoreKit IAP.** No QR codes, license keys, AR markers, or direct Stripe links (outside allowed exceptions).

**External purchase links — storefront-dependent (frequently misunderstood):**

| Storefront | External CTA allowed? | Entitlement needed? |
|-----------|----------------------|-------------------|
| United States | Yes | No (as of current guidelines) |
| EU (iOS/iPadOS) | With StoreKit External Purchase Link Entitlement | Yes |
| Japan (Japan App Store storefront) | **Alternative Payment Processing** and/or **Out-of-App Offers** only with Apple-granted entitlement + **StoreKit** eligibility/disclosure flows; see ADPLA **Attachment 12** and Apple Materials | Yes |
| Netherlands dating apps | With specific entitlement | Yes |
| South Korea | With StoreKit External Purchase Entitlement | Yes |
| All others | No | N/A |

**Japan (Attachment 12) — design rules that differ from “US-style” external links:** IAP must be offered **at least as prominently** as any alternative payment option on the same screen; **Out-of-App** purchases use an **actionable link** that opens the **default web browser** (not an in-app `WKWebView`); no website-purchase promo on the **App Store product page**; **Kids** category and age rules block certain offers; entitlement/device OS requirements are defined in Apple Materials (ADPLA references **iOS 26.2+**, Japan devices). Full checklist: [ADDENDUM.md](ADDENDUM.md#japan-app-store-adpla-attachment-12-commerce--distribution).

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

**Intellectual property, Apple marks & third-party brands (§5.2, ADPLA §3.1(d), §3.3.4)**  
- You **certify** under the ADPLA that the app, **App Store metadata** (screenshots, description, previews, icons), **Xcode Cloud** content you provide, and **Wallet Pass** materials do **not** infringe **Apple’s or anyone else’s** rights — including **copyright, trademark, trade secret, patent, right of publicity, music / performance / sync licenses, video and stock-photo rights, fonts, and third-party data / API terms**.  
- **App content**: only assets **you own** or **properly license** (including user-generated or partner content you display).  
- **Third-party logos / names** in UI, notifications, passes, or marketing: use only with **permission** and in ways that **don’t imply endorsement** or confuse origin (**§5.2**, **§5.2.4**).  
- **Apple marks & product imagery** (App Store logo, iPhone silhouettes, “Compatible with…,” **Apple Pay** marks, **Wallet** badges, screenshots of Apple UI): follow Apple’s **written** marketing and identity rules — not informal reuse. Primary entry points:  
  - [Marketing Resources and Identity Guidelines](https://developer.apple.com/app-store/marketing/guidelines/)  
  - [Guidelines for Using Apple Trademarks and Copyrights](https://www.apple.com/legal/intellectual-property/guidelinesfor3rdparties.html) (also cited in ADPLA **§2.6** for referencing Apple products)  
  - [Apple Pay Marketing Guidelines](https://developer.apple.com/apple-pay/marketing/) · [Add to Apple Wallet Guidelines](https://developer.apple.com/wallet/add-to-apple-wallet-guidelines/)  
- **§5.2.1–5.2.3**: no **protected third-party material** without permission; no **misleading or copycat** representations; honor **third-party service terms** if you access or monetize their content; do not facilitate **illegal file sharing** or unauthorized **download/save** of media from third-party sources (e.g. streaming services) unless expressly permitted.  
- **§5.2.4 Apple endorsements**: do **not** suggest or imply Apple **supplies**, **endorses**, or **guarantees** your app’s quality. **“Editor’s Choice”** (and similar) badges are **applied only by Apple**.  
- **§5.2.5 Apple products**: do not make an app **confusingly similar** to Apple’s **products, interfaces, App Store / iTunes / Messages**, or **Apple ads** — includes look-and-feel that mimics system or store UI.  
- **Disputes**: third-party IP claims against your app are handled via Apple’s [IP dispute process](https://www.apple.com/legal/intellectual-property/dispute-forms/index.html) when applicable.  
- **Safari / Website Push IDs (ADPLA Attachment 1 §3)**: notifications must be sent under **your** site’s **name and brand**; include **your** icon / logo / identifying mark; **do not impersonate** another site or mislead about the sender. If you reference a **third-party** trademark in a push, you **warrant** you have the **rights**. Enabling Website Push permits Apple to use **screenshots** of those notifications and **your** associated marks in **Apple marketing**, except portions you **identify in writing** as not cleared for promotion.  
- **In-app Apple services** (e.g. **MusicKit**, **ShazamKit**): follow **Apple Music Identity Guidelines** and program docs — artwork and metadata often **cannot** be split from permitted playback/UI patterns.  
- Reference: [§5.2 Intellectual Property](https://developer.apple.com/app-store/review/guidelines/#legal) (scroll **Legal** section)

**App Clips / Widgets / Extensions (often overlooked):**
- App Clips cannot include advertising.
- Widgets, extensions, and notifications must be directly related to app content/functionality.
- Siri intents must match what the app actually does; avoid registering irrelevant intents.

**Reference:** [developer.apple.com/app-store/review/guidelines/#minimum-functionality](https://developer.apple.com/app-store/review/guidelines/#minimum-functionality) · [Legal (IP, location)](https://developer.apple.com/app-store/review/guidelines/#legal)

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

## Gate K — Safety, speech, gambling, sandbox & system access

**Apple anchors:** [Safety (§1.x)](https://developer.apple.com/app-store/review/guidelines/#safety) · [Legal — §5.3 Gaming / gambling](https://developer.apple.com/app-store/review/guidelines/#legal) · [Software requirements / sandbox expectations](https://developer.apple.com/app-store/review/guidelines/#software-requirements)  
**Deeper dives:** [REFERENCE.md — Gaming & gambling](REFERENCE.md#gaming-gambling-and-lotteries-53) · [EDGE-CASES — Content policy](EDGE-CASES.md#content-policy-edge-cases)

### Violence, weapons & objectionable content (§1.1)

- Apple rejects **offensive, gratuitous, or harmful** content — including realistic **killing, torture, or abuse** of people or animals when presented to **shock** or **glorify** harm (see full **§1.1** text for examples and nuance).
- **§1.1.3 — Weapons:** Apps must not **encourage illegal or reckless** use of weapons/dangerous objects or **facilitate purchase** of **firearms/ammunition** where the guideline prohibits it. Hunting/sport contexts still need **appropriate age rating** and compliance with local law.
- **Games:** Fictional “enemies” must not **solely target** a **real** race, culture, government, corporation, or other real-world group (**§1.1.2**).
- **Terrorism / serious harm:** Content that **threatens, incites, or promotes violence** or **CSAM** is prohibited under guidelines and **ADPLA §3.2(b)** — immediate legal/safety escalation, not a routine review tweak.

### “Open speech,” politics, satire & forums

- The App Store is a **private platform**: Apple enforces **§1.1** (e.g. hate, harassment, defamation) and local-law-sensitive material. **First Amendment / “free speech”** doctrines **do not** bind Apple’s acceptance of your app.
- **§1.1.1** explicitly notes that **professional political satirists and humorists** are **generally exempt** from a narrow part of the objectionable-content rule — **satire used as cover for harassment** or **defamation** still fails review.
- **UGC / social / debate apps:** You still need **§1.2** tooling (filter, report, block, contact, timely response) and **age-appropriate** controls; “open discussion” is not a bypass for **illegal** content or **missing moderation**.

### Gambling, contests & loot boxes (§5.3 + §3.1.1)

- **Real-money gaming** (sportsbook, casino, poker, etc.): **licensing** in every jurisdiction you ship; **geo-restrict** to licensed regions; app must be **free to download**; **no IAP** to purchase **gambling credits** (**§5.3**). Expect **extra review** time.
- **Sweepstakes / contests:** Must be **run by you**; **official rules** in-app must state **Apple is not a sponsor** (**§5.3.1–5.3.2**).
- **Loot boxes / randomized digital items:** Disclose **odds** of item types **before purchase** (**§3.1.1**).
- **Gray areas** (skill contests, fantasy, prediction markets): **legal review** per territory — see [EDGE-CASES — Gambling gray zone](EDGE-CASES.md#gambling-gray-zone).

### Sandboxing, containers & “system access”

- **iOS / iPadOS / tvOS / watchOS / visionOS:** Apps must stay within Apple’s **documented** model: your **sandbox / designated container**, **extensions** only in **documented** surfaces, **no private APIs** to poke other apps’ data or system internals (**§2.5.1**, **ADPLA §3.3.1**). Use **public** APIs for sharing (share sheet, `UIDocumentPicker`, App Groups where appropriate, etc.).
- **Mac App Store:** **Sandbox required** per **§2.4.5** — no third-party installers, no escaping the store for updates, no **root/setuid** — see [REFERENCE.md — Mac App Store](REFERENCE.md#mac-app-store-additional-rules-245).
- **Privileged capabilities** (examples): **VPN** (**§5.4** — org account, `NEVPNManager`, strict no-resale-of-user-data), **MDM** (**§5.5** — enterprise/gov/edu patterns, Apple approval), **Network Extension** (declared networking purpose only; see **Gate C**). Misusing these for **surveillance**, **ads**, or **undeclared** data collection → rejection and program risk.
- **Settings / system UI:** Do not deep-link into arbitrary **Settings** panes with **private URL schemes** (`prefs:root`, etc.); open **your app’s** settings via **`openSettingsURLString`** (**Gate F**).
- **macOS “power user” APIs** (Accessibility, Automation, screen capture, input monitoring): Use **only** for **stated, legitimate** features; reviewers may flag **Automation** or **Accessibility** used to **control other apps** for non–a11y purposes. Document the **user-visible** purpose in **Notes for Review** and match **entitlements** to reality.

**Reference:** [App Review Guidelines — Safety](https://developer.apple.com/app-store/review/guidelines/#safety)

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

Hardware (if any): [Connection type + device name; confirm App Store Connect §6.1 disclosure matches]

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

- **[REFERENCE.md](REFERENCE.md)** — Full rejection database with community case studies, Apple doc anchors, Reddit/StackOverflow patterns, fix code examples; **§5.2 / §5.1.5 / ads–push / export** pattern tables; **§5.3 gambling**, **§1.1 safety/speech**, **sandbox & system access** tables
- **[EDGE-CASES.md](EDGE-CASES.md)** — AI policy, hot updates, privacy manifests, external payments, UGC edge cases, regulated domains
- **[CHECKLIST.md](CHECKLIST.md)** — Printable pre-submission checklist and reviewer kit templates
- **[ADDENDUM.md](ADDENDUM.md)** — Vibe coding crackdown (March 2026), age rating overhaul, visionOS requirements, app size limits, ITMS error codes, ATS exceptions, launch screen, code signing, TestFlight vs App Store review, inactive app removal, screenshot requirements, local network permission, notification timing, post-approval quality (§5.6.4), **Japan ADPLA Attachment 12** (alternative payments / out-of-app / marketplace)

**Maintenance:** Rejection statistics (e.g. 2024 transparency counts) should be refreshed when Apple publishes new annual reports; gate logic and guideline links stay authoritative until Apple updates the guidelines page.
