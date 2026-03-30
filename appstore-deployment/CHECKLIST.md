# App Store Pre-Submission Checklist & Templates

Copy and use these checklists before every submission.

---

## Master Pre-Submission Checklist

### Gate A — Hard Blockers (Fix First)

- [ ] Privacy policy URL returns HTTP 200 (not a redirect to a login wall)
- [ ] Support URL returns HTTP 200
- [ ] **Xcode → File → Generate Privacy Report** — no missing manifests for [listed SDKs](https://developer.apple.com/support/third-party-SDK-requirements/); resolve `ITMS-91061`-class issues before upload
- [ ] **Required reason APIs** — If app or embedded targets use covered APIs (disk space, file timestamps, UserDefaults, boot time, active keyboard, etc.), `PrivacyInfo.xcprivacy` on **that target** includes `NSPrivacyAccessedAPITypes` with [approved reasons](https://developer.apple.com/documentation/bundleresources/privacy_manifest_files/describing_use_of_required_reason_api)
- [ ] **App Store Connect → Validate App** (or upload dry-run) — no privacy-manifest or signature errors reported
- [ ] **EU DSA** — Trader status and verified contact flow completed if distributing in the EU as a trader; hobbyist/non-trader path declared if applicable ([ASC Help](https://developer.apple.com/help/app-store-connect/manage-compliance-information/manage-european-union-digital-services-act-trader-requirements/))
- [ ] Export compliance set in Info.plist (`ITSAppUsesNonExemptEncryption`)
- [ ] App not using `UIWebView` (deprecated; use `WKWebView`)
- [ ] No private Settings URL schemes (`prefs:root`, `App-Prefs`, etc.) — use `UIApplication.openSettingsURLString` for your app's settings
- [ ] Quick scan: `strings YourApp | grep -E 'prefs:root|App-Prefs'` returns nothing suspicious
- [ ] Code signing: entitlements match provisioning profile and App ID capabilities (debug with `codesign -d --entitlements :- YourApp.app`)
- [ ] LaunchScreen storyboard present — verify `UILaunchStoryboardName` in Info.plist (required since April 2020)
- [ ] Screenshots: exact pixel dimensions for all targeted device sizes (1px off fails upload)
- [ ] App binary under 4 GB total; watchOS under 75 MB; aim under 200 MB for cellular download conversion
- [ ] ATS: no global `NSAllowsArbitraryLoads` without documented justification; prefer domain-specific `NSExceptionDomains`
- [ ] AASA file (if using Universal Links): accessible without auth/redirects at `/.well-known/apple-app-site-association`
- [ ] visionOS: app motion declaration completed in ASC (required before submission)
- [ ] Age rating questionnaire re-answered for current build content (new 13+/16+/18+ categories since July 2025)

### Gate B — App Completeness

- [ ] App tested on real device via TestFlight (not only Simulator)
- [ ] Zero crashes on: cold start, onboarding, login, core feature, paywall, checkout
- [ ] No placeholder/Lorem ipsum content anywhere in the UI
- [ ] No version strings like "0.1", "beta", "alpha" in app name or UI
- [ ] All in-app purchases submitted with the binary and status = "Ready to Submit"
- [ ] IAP products visible and purchasable in the TestFlight/production environment
- [ ] Demo account created, tested, and entered in App Review Information
- [ ] Backend services accessible from external network (no VPN required)
- [ ] All deep-link URLs and universal links tested
- [ ] App works on IPv6-only network (test via Mac Internet Sharing hotspot)

### Gate C — Privacy & Permissions

- [ ] Privacy policy link present inside the app (Settings or About screen)
- [ ] Privacy policy link entered in App Store Connect metadata field
- [ ] App Privacy nutrition label completed and matches actual data flows
- [ ] Every permission request has a clear, user-benefit purpose string in Info.plist
- [ ] No permission required to access core functionality (permission denied = degraded feature, not blocked app)
- [ ] ATT prompt implemented if app does any cross-app or cross-site tracking
- [ ] Third-party AI providers disclosed in privacy policy and App Privacy label
- [ ] Account deletion: in-app initiated, deletes account + data, easy to find
- [ ] No mandatory login for features that don't require identity

### Gate D — Payments

- [ ] All digital feature/content unlocks use StoreKit IAP
- [ ] No external purchase CTAs (unless US storefront and current guidelines allow)
- [ ] Subscription paywall shows: price, billing period, auto-renewal disclosure, trial terms
- [ ] The **amount actually billed** is the most prominent pricing element in the paywall
- [ ] "Restore Purchases" button present in subscription/paywall UI
- [ ] In-app path exists to manage/cancel subscriptions (system UI or clear help path)
- [ ] Paywall loads products correctly from a fresh install (tested on clean device)
- [ ] Receipt validation uses production-first, sandbox-fallback pattern
- [ ] No charging for built-in OS capabilities (push notifications, iCloud storage, camera)
- [ ] No fake scarcity, misleading countdowns, or visually deceptive trial emphasis

### Gate E — Design & Metadata

- [ ] **Age rating questionnaire** in App Store Connect matches real content (recheck after adding social, UGC, or creator tools)
- [ ] Primary flows pass a **VoiceOver** smoke test (labels, focus order)
- [ ] Screenshots match current build's actual UI
- [ ] App description matches current features (no removed features still advertised)
- [ ] "What's New" is specific (not "bug fixes and performance improvements")
- [ ] App is not a thin WebView wrapper without native functionality
- [ ] No multiple Bundle IDs for near-identical apps
- [ ] All features accessible via normal navigation (no hidden features)
- [ ] New features documented in Notes for Review with navigation path
- [ ] App Clip (if present) contains no advertising
- [ ] Widgets/extensions/notifications are related to app functionality
- [ ] Siri intents/Shortcuts registered only for tasks the app actually performs

### Gate F — Code Execution

- [ ] No hot-update frameworks downloading new executable code
- [ ] React Native/Expo: OTA updates do not change functionality post-review
- [ ] No `eval()` of remotely fetched code
- [ ] Feature flags only show/hide existing features (don't introduce new code paths)
- [ ] Educational code sandbox: code is user-viewable and user-editable per §2.5.2 exception

### Mac App Store only (if distributing on MAS)

- [ ] Sandboxed per §2.4.5; no third-party installer
- [ ] No license-key or license screen at launch; updates only via Mac App Store
- [ ] No Sparkle or other external updater bundled in the MAS build
- [ ] No auto-login items / background daemons without explicit user consent

### Gate G — AI & UGC

- [ ] AI features: user consent obtained before sending personal data to third-party AI
- [ ] AI-generated user-visible content: report, block, filter mechanisms present
- [ ] UGC moderation: filter, report (with timely response), block, contact info — all four present
- [ ] NSFW content: hidden by default; enable toggle only available on your website (not in-app)
- [ ] Kids Category: no third-party analytics, no third-party advertising, no tracking identifiers
- [ ] If the app is a creator / AI content app, age-rating answers in App Store Connect were re-reviewed for 2026 updates
- [ ] If content can exceed the app age rating, underage access is restricted using declared or verified age

### Gate H — Trust / Anti-Scam

- [ ] No forced rating/review prompts to unlock access
- [ ] No forced downloads of other apps or social-posting tasks to unlock access
- [ ] No permissions gated as a condition of use when the feature can degrade gracefully
- [ ] No misleading copy around pricing, discounts, urgency, or trial conversion
- [ ] No claims in marketing/screenshots that the app cannot substantiate in the reviewed build
- [ ] The submitting entity clearly matches the regulated service provider, if applicable

### Gate I — Review Operations & Evidence

- [ ] Resolution Center packet prepared: issue clause + repro steps + fix summary
- [ ] Reviewer-only diagnostics enabled for auth/IAP/network failures (non-PII)
- [ ] Correlation IDs included in backend logs for review build requests
- [ ] Screenshots/video attachment prepared for non-obvious or hardware-specific flows
- [ ] If rejection is metadata-only, plan same-build resubmission path
- [ ] If appealing, draft one evidence-based appeal (one per rejected submission)

---

## App Review Notes Template

```
==========================================================
APP REVIEW NOTES — [App Name] v[Version]
==========================================================

WHAT THIS APP DOES:
[2-3 sentences. What problem does it solve? Who uses it?]

TARGET AUDIENCE:
[Describe primary users: e.g., "iOS developers who want to test their apps"]

DEMO ACCOUNT:
  Email: [email address]
  Password: [password]
  2FA: Disabled for this account
  Region/Storefront: [e.g., United States]
  Pre-loaded data: [describe any data the reviewer needs to see features]

STEP-BY-STEP KEY FLOW:
  1. Launch app → [what happens]
  2. Tap "[Button]" → [what it does]
  3. [Core feature] is accessible via [exact path]
  4. [Paywall/subscription] appears when: [exact trigger]
     - Tap [X] to see the paywall
     - Test purchase with sandbox account
  5. Account deletion: Settings → Account → Delete Account

IN-APP PURCHASES INCLUDED:
  - [Product ID]: [Description] — [Price Tier]
  - [Product ID]: [Description] — [Price Tier]
  Note: Products are submitted with this binary and are in "Ready to Submit" state.

AI FEATURES (if applicable):
  - [Feature name] uses [Provider] for [specific function]
  - User consent is requested before data is transmitted
  - On-device inference: [Yes/No]
  - Data retained by provider: [Yes/No — if Yes, disclosed in privacy policy]

AGE / SAFETY CONTROLS (if applicable):
  - App age rating in App Store Connect: [e.g. 17+]
  - Content exceeding age rating can be flagged by users: [Yes/No]
  - Underage access restriction method: [declared age / verified age / none]
  - NSFW controls: [hidden by default / website-side toggle / not applicable]

NON-OBVIOUS BEHAVIORS:
  - [Describe any feature that might look like a bug but is intentional]
  - [Describe any feature only accessible after specific actions]
  - [Describe any feature gated on user state/data]

BACKEND STATUS:
  All backend services are live and accessible without VPN from any network.

HARDWARE/SPECIAL REQUIREMENTS:
  [None / or describe: "Requires Bluetooth LE hardware; video attached"]

ATTACHMENT:
  [Link to demo video if applicable, or "No attachment"]

CONTACT:
  Developer: [Name]
  Email: [email]
  Phone: [optional]
==========================================================
```

---

## Demo Account Package Template

```
DEMO ACCOUNT PACKAGE
====================
Username: demo@yourapp.com
Password: DemoAccount123!
2FA: Not enabled on this account
Region: United States (App Store storefront: US)

Pre-seeded state:
- Profile: [Jane Doe, sample avatar, bio filled]
- [Feature A]: [sample data pre-loaded]
- [Feature B]: [sample content visible]
- Subscription: [Active monthly subscription for testing premium features]
  OR
  Subscription: [Not active — reviewer can trigger paywall via Settings → Upgrade]

Guest mode: [Available — tap "Continue without account" on login screen]

To test account deletion:
  Settings → Account → Delete Account
  (Account is resettable; email support@yourapp.com to restore for repeat testing)

Backend: https://api.yourapp.com — live, no VPN needed
```

---

## Rejection Response Templates

### Template A — App Completeness (2.1) Response

```
Thank you for reviewing [App Name].

We've addressed the issue you identified:

[Specific issue]: [Specific fix applied]

The demo account credentials in the App Review Information section have been 
verified and allow full access to all features including [specific feature].

The in-app purchases are now submitted alongside this build in "Ready to Submit" 
state and should be visible to reviewers.

Navigation path to reach [specific feature]:
1. [Step 1]
2. [Step 2]
3. [Result]

We've resubmitted the build with these changes and look forward to your review.
```

### Template B — Privacy/Permission Response

```
Thank you for your review of [App Name].

Regarding Guideline 5.1.1: [Specific concern]

We have made the following changes:
1. [Updated privacy policy URL to: https://yourapp.com/privacy]
2. [Added accessible privacy policy link in app: Settings → Privacy Policy]
3. [Updated purpose string for [permission] to clearly state: "[new string]"]
4. [Added account deletion at: Settings → Account → Delete Account]

The purpose string for [permission] now reads:
"[Exact new purpose string that clearly explains user benefit]"

Our privacy policy at [URL] now explicitly covers:
- Data collected: [list]
- Data not collected: [list]  
- Third parties: [list, or "None"]

We believe these changes fully address the concern raised.
```

### Template C — Payments/IAP Response

```
Thank you for reviewing [App Name].

Regarding Guideline 3.1.1: [Specific concern]

[App Name] is a [multiplatform service / enterprise service / free companion app] 
that [describe qualifying category].

[Choose applicable]:

OPTION A (multiplatform service):
Users subscribe via our website and receive access across all platforms including 
iOS, Android, and web. The iOS app provides access to content users have already 
purchased on other platforms. We do not include any call-to-action directing users 
to purchase in the app. This qualifies as a multiplatform service per §3.1.3(b).

OPTION B (free companion app):
The iOS app is entirely free and serves as a companion to our web platform. Users 
manage their subscription on the web. We have removed any reference to pricing or 
external purchase within the app binary. This qualifies per §3.1.3(f).

OPTION C (IAP added):
We have implemented StoreKit in-app purchase for [feature/subscription]. The 
paywall is now accessible at [path] and products are submitted with this build.
```

### Template D — Metadata/Hidden Features Response

```
Thank you for reviewing [App Name].

Regarding Guideline 2.3.1:

We have updated the Notes for Review with complete documentation of all features:

[Feature that was flagged]:
- Purpose: [explanation]
- How to access: [exact navigation path]
- Why it exists: [user benefit]
- It is NOT a hidden feature; it is accessible via [path] and documented in 
  the app's [Help section / Onboarding / Settings]

We have also updated [screenshots/description] to reflect [specific change].

The updated Notes for Review with complete feature documentation are attached 
to this submission.
```

### Template E — Trust / Manipulative Practices Response

```
Thank you for the review of [App Name].

Regarding the concern under [Guideline 3.1.2 / 5.6 / 3.2.2]:

We have revised the affected flow to remove any potentially misleading or 
manipulative elements:

1. [Example: The billed amount is now the most prominent pricing element]
2. [Example: We removed countdown copy that could imply false scarcity]
3. [Example: We added a clear Restore Purchases / Manage Subscription path]
4. [Example: We removed any requirement to rate the app or enable tracking]

The updated flow can be reviewed at:
[exact navigation path]

We believe these changes address the customer-trust concern raised in review.
```

### Template E — Reviewer-Only Login/Purchase Failure Response

```
Thank you for the review and for flagging this issue.

We investigated this in the review build and added temporary diagnostics to
capture the exact failure path. We identified the issue as:
[e.g., token refresh failure on first login / regional backend route mismatch /
IAP product state mismatch].

Fix applied:
1. [Specific code or backend fix]
2. [Any config/environment fix]
3. [Validation performed on physical TestFlight devices]

To verify in this submission:
1. [Exact path]
2. [Expected result]

We also attached [screenshot/video] and updated App Review Notes with the
exact walkthrough.
```

---

## Subscription Paywall Compliance Checklist

Specifically for §3.1.2 compliance:

- [ ] **Subscription price** is displayed prominently (not in small gray text)
- [ ] **Actual billed amount** is the most prominent price on screen
- [ ] **Billing period** is clear ("per month", "per year", not just "monthly")
- [ ] **Free trial duration** shown in same visual prominence as paid period terms
- [ ] **Auto-renewal disclosure** present: "Subscription auto-renews unless canceled"
- [ ] **Cancel instructions**: link to App Store subscriptions management
- [ ] **Restore Purchases** button accessible (often required in Settings or paywall footer)
- [ ] **Subscription name + service provided** are visible on sign-up screen
- [ ] **Trial terms**: exact duration + what changes after trial ends
- [ ] **Price comparison** (if showing before/after pricing): must be accurate and not misleading
- [ ] Privacy Policy link accessible from paywall or app settings
- [ ] Terms of Use link in App Store description (ASC doesn't have a ToU field; put URL in description text)

**Compliance example layout:**

```
┌─────────────────────────────────────┐
│           Premium Access            │
│                                     │
│  ✓ Feature A                        │
│  ✓ Feature B                        │
│  ✓ Feature C                        │
│                                     │
│  ┌─────────┐  ┌─────────────────┐   │
│  │ Monthly │  │    Annually     │   │
│  │ $9.99   │  │ $59.99 ($4.99/  │   │
│  │ /month  │  │  month, save   │   │
│  └─────────┘  │    50%)         │   │
│               └─────────────────┘   │
│                                     │
│  7-day free trial, then $9.99/month │ ← Required disclosure
│  Cancel anytime in App Store        │
│                                     │
│  [ Start Free Trial ]               │
│                                     │
│  Auto-renews. Cancel anytime before │ ← Required
│  trial ends. Restore | Privacy      │
└─────────────────────────────────────┘
```

---

## Supply Chain Check Script

```bash
#!/bin/bash
# privacy-check.sh — Run before archive

SCHEME="$1"
BUNDLE_ID="$2"

echo "🔍 Running pre-submission privacy manifest check..."

# Build archive
xcodebuild archive \
    -scheme "$SCHEME" \
    -archivePath /tmp/check.xcarchive \
    -destination 'generic/platform=iOS' \
    -quiet

# Generate privacy report
echo "📋 Generating privacy report..."
# In Xcode GUI: Product → Archive → Validate App → Review Privacy Report
# Look for: "Missing privacy manifest" warnings

# Check for common problematic patterns
echo "🔎 Checking for known rejection patterns..."

# Private Settings URL schemes
grep -rE "prefs:root|App-Prefs" "$SRCROOT" --include="*.swift" --include="*.m" && \
    echo "❌ Found private Settings URL scheme (use openSettingsURLString)"

# UIWebView
grep -r "UIWebView" "$SRCROOT" --include="*.swift" --include="*.m" && \
    echo "❌ Found deprecated UIWebView"

# Hardcoded IPs (rough check)
grep -rE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' "$SRCROOT" --include="*.swift" \
    --include="*.m" --include="*.plist" | \
    grep -v "127.0.0.1\|0.0.0.0\|255.255.255" && \
    echo "⚠️  Possible hardcoded IPv4 address"

# Hot update indicators
grep -r "CodePush\|rollout\|jspatch\|JSPatch" "$SRCROOT" --include="*.swift" \
    --include="*.m" --include="*.podspec" --include="Podfile" && \
    echo "❌ Hot-update framework detected"

echo "✅ Check complete. Review warnings above."
```

---

## Quick Reference: Reviewer Message → Guideline → Fix

| Reviewer says | Guideline | Fix |
|--------------|-----------|-----|
| "App crashed during review" | 2.1(a) | Test on device; fix crash; resubmit |
| "Cannot locate in-app purchase" | 2.1(b) | Submit IAP with binary; ensure "Ready to Submit" |
| "App requires login for non-account features" | 5.1.1(v) | Add guest mode or remove login gate |
| "Account deletion not found" | 5.1.1(v) | Add in-app delete account flow |
| "Privacy policy link is broken / missing" | 5.1.1(i) | Fix URL; add in-app link |
| "Purpose string insufficient for [permission]" | 5.1.1 | Rewrite as user-benefit, not app-benefit |
| "App is similar to a website" | 4.2 | Add native features; explain in notes + attach video |
| "App may contain hidden features" | 2.3.1 | Document all features in Notes for Review |
| "External purchase mechanism found" | 3.1.1 | Remove CTA or implement IAP |
| "Subscription info not clear" | 3.1.2 | Make price + period prominent; add disclosure |
| "No Sign in with Apple" | 4.8 | Add SiwA alongside existing third-party login |
| "Non-public API detected" | 2.5.1 | Replace private API with public alternative |
| "App downloads executable code" | 2.5.2 | Remove hot-update; restrict to content updates |
| "UGC moderation insufficient" | 1.2 | Add report, block, filter, contact info |
| "Missing privacy manifest" | ITMS-91061 | Update SDK to version with manifest |
| "Personal data shared without consent" | 5.1.2 | Add consent before AI/analytics data transmission |
| "Paywall is misleading / amount not prominent" | 3.1.2 / 5.6 | Make billed amount dominant; remove deceptive emphasis |
| "App requires store-related action for access" | 3.2.2(x) | Remove forced ratings/downloads/reviews |
| "Age rating or creator safeguards insufficient" | 1.2 / age-rating requirements | Add flagging + age restriction mechanism; update ASC answers |
| Submission blocked in ASC (no guideline clause) | EU DSA trader / compliance | Complete trader verification or declare non-trader; [ASC DSA Help](https://developer.apple.com/help/app-store-connect/manage-compliance-information/manage-european-union-digital-services-act-trader-requirements/) |
| Email about required-reason APIs / upload fails | Privacy manifest § required reasons | Add `NSPrivacyAccessedAPITypes` to the bundle that contains the code using covered APIs |
| "Invalid provisioning profile" | ITMS-90161 | Regenerate profile with valid cert and matching capabilities |
| "Invalid binary" | ITMS-9000 | Check entitlements, metadata, signing, SDK version |
| "Entitlements not in provisioning profile" | Signing | Enable capability in Dev Portal; regenerate profile |
| "App introduces or changes features via code" | 2.5.2 | Remove vibe coding / code execution; see ADDENDUM.md |
| "Launch screen not found" | Build requirement | Add LaunchScreen storyboard; set UILaunchStoryboardName |
| "Age rating inconsistent with content" | 2.3 / Age rating | Re-answer questionnaire; use developer override for COPPA |
| "App motion information required" | visionOS submission | Complete app motion declaration in App Store Connect |
| NSAllowsArbitraryLoads justification needed | ATS / 5.1 | Switch to domain-specific exceptions; document rationale |
