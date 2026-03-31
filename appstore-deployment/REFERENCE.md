# App Store Rejection Reference Database

Complete rejection patterns with Apple documentation anchors, community case studies, and specific fixes.  
**Authority**: Apple's App Review Guidelines (last updated February 6, 2026)  
**Stats**: Apple reviewed 7,771,599 submissions in 2024; rejected ~1,931,400 (~24.9%).

---

## Ranked Rejection Database

### #1 — Guideline 2.1: App Completeness (Performance Section)

**Apple anchor:** https://developer.apple.com/app-store/review/guidelines/#app-completeness  
**Apple quote:** *"Over 40% of unresolved issues are related to Guideline 2.1."*  
**Probability: 5/5 | Fix impact: 4/5**

#### Sub-causes and specific fixes

| Sub-cause | Reviewer message pattern | Fix |
|-----------|--------------------------|-----|
| Crash on launch or during core flow | "App crashed when I tapped [X]" | Test on device via TestFlight; use Instruments to catch memory/threading issues |
| Placeholder content | "App contains Lorem ipsum / placeholder images" | Grep codebase for "lorem", "placeholder", "TODO" strings before submission |
| Missing demo account | "App requires login but no demo credentials provided" | Create a pre-seeded account; disable 2FA for that account; add to App Review Information in ASC |
| Backend not accessible | "Cannot connect to server during review" | Ensure staging/prod environment is reachable without VPN from external networks |
| IAP products not visible | "Cannot locate the in-app purchase" | Submit IAPs together with the binary; ensure products are in "Ready to Submit" state |
| Broken support/privacy URLs | "Link in app returns 404" | Automate URL health checks as pre-submission step |
| Broken features on first launch | "Continue button does nothing" | Cold-start test on a fresh install on a physical device with the production build |

#### IAP Catch-22 (very common loop)
**Problem:** Developer creates IAP in ASC but doesn't submit it with the binary.  
**Symptom:** Blank paywall, or "Continue" button unresponsive during review.  
**Fix sequence:**
1. In ASC → Your App → In-App Purchases → select the product → set status to "Ready to Submit"
2. In the submission form, check "Submit this in-app purchase for review" alongside the binary

**Receipt validation — Apple-approved pattern:**
```swift
// Production-first, sandbox fallback
func validateReceipt(data: Data) async throws -> ReceiptResponse {
    do {
        return try await verifyWith(endpoint: .production, receipt: data)
    } catch ReceiptError.sandboxReceiptOnProduction {
        return try await verifyWith(endpoint: .sandbox, receipt: data)
    }
}
```

**Community sources:**  
- RevenueCat guide: https://www.revenuecat.com/blog/growth/the-ultimate-guide-to-app-store-rejections/  
- Reddit r/iOSProgramming (search "2.1 rejection IAP")  
- StackOverflow: https://stackoverflow.com/questions/tagged/app-store-rejection+in-app-purchase  

---

### #2 — Guideline 5.1.1: Privacy Policy & Data Disclosure (Legal Section)

**Apple anchor:** https://developer.apple.com/app-store/review/guidelines/#privacy  
**Probability: 4/5 | Fix impact: 4/5**

#### Required artifacts (all three must agree)

1. **In-app accessible privacy policy** — Settings/About screen minimum; must be tappable link
2. **App Store Connect metadata** — Privacy Policy URL field in ASC
3. **App Privacy nutrition label** — Data types collected, purposes, linked/not-linked to user

#### Purpose string requirements (per-permission)

```xml
<!-- Info.plist — every key must have a user-benefit string, not a developer-benefit string -->

<!-- Camera -->
<key>NSCameraUsageDescription</key>
<string>Scan QR codes to join sessions — your camera is not recorded or stored.</string>

<!-- Location (when in use) -->
<key>NSLocationWhenInUseUsageDescription</key>
<string>Your location shows nearby events on the map.</string>

<!-- Contacts -->
<key>NSContactsUsageDescription</key>
<string>Find friends who are already on [App] to connect with them.</string>

<!-- Microphone -->
<key>NSMicrophoneUsageDescription</key>
<string>Record voice notes attached to your tasks.</string>

<!-- Photos -->
<key>NSPhotoLibraryUsageDescription</key>
<string>Choose a photo for your profile picture or share images in posts.</string>

<!-- Photo saving (separate from reading) -->
<key>NSPhotoLibraryAddUsageDescription</key>
<string>Save generated artwork to your Photos library.</string>

<!-- Tracking (ATT) -->
<key>NSUserTrackingUsageDescription</key>
<string>This helps us show you more relevant ads and measure campaign effectiveness.</string>

<!-- Bluetooth (always — e.g. BLE peripherals) -->
<key>NSBluetoothAlwaysUsageDescription</key>
<string>Bluetooth stays on in the background so [App] can stay connected to your [device name].</string>

<!-- Motion & fitness -->
<key>NSMotionUsageDescription</key>
<string>Motion data is used to count steps and detect activity for your daily summary.</string>

<!-- Speech recognition -->
<key>NSSpeechRecognitionUsageDescription</key>
<string>Speech is converted to text so you can dictate notes without typing.</string>

<!-- Face ID / Touch ID (when gating with LocalAuthentication) -->
<key>NSFaceIDUsageDescription</key>
<string>Unlock the app and sign in faster using Face ID.</string>
```

**Rejection pattern:** Generic purpose string  
Bad: `"We need your location."`  
Good: `"Your location is used to show coffee shops near you on the map."`

#### Account deletion (required since June 30, 2022)

**Apple doc:** https://developer.apple.com/support/offering-account-deletion-in-your-app/

Required behavior:
- In-app initiated (not email to support)
- Must delete account AND all associated personal data
- Data legally required to be retained may be exempted but must be disclosed
- Account deletion flow must be easy to find (not buried)

```swift
// Example: Presenting account deletion in SwiftUI
struct AccountSettingsView: View {
    @State private var showDeleteConfirmation = false
    
    var body: some View {
        Section("Danger Zone") {
            Button(role: .destructive) {
                showDeleteConfirmation = true
            } label: {
                Label("Delete Account", systemImage: "person.crop.circle.badge.minus")
            }
        }
        .confirmationDialog(
            "Delete your account?",
            isPresented: $showDeleteConfirmation,
            titleVisibility: .visible
        ) {
            Button("Delete Account", role: .destructive) {
                Task { await deleteAccount() }
            }
        } message: {
            Text("This will permanently delete your account and all associated data. This cannot be undone.")
        }
    }
}
```

**Community sources:**  
- Adapty blog: https://adapty.io/blog/app-store-rejection/  
- Apple support: https://developer.apple.com/support/offering-account-deletion-in-your-app/

---

### #3 — Guideline 3.1.1: In-App Purchase Required (Business Section)

**Apple anchor:** https://developer.apple.com/app-store/review/guidelines/#payments  
**Probability: 3/5 | Fix impact: 5/5**

#### What MUST use IAP

- Subscription access (premium features, content tiers)
- In-game currency, gems, coins
- Unlocking app capabilities or full version
- Access to premium content catalogs
- Digital gift cards / vouchers redeemable for digital content
- Loot boxes (must also disclose odds before purchase)

#### What is EXEMPT from IAP

| Category | Rules | Apple doc |
|----------|-------|-----------|
| Reader apps | Previously-purchased media catalogs; can link to website via entitlement | https://developer.apple.com/support/reader-apps/ |
| Multiplatform services | Access to content purchased on another platform; no purchase CTA in-app | §3.1.3(b) |
| Enterprise/education | Sold in bulk to organizations; not family purchases | §3.1.3(c) |
| Person-to-person services | Real-time 1:1 (tutors, trainers, doctors) | §3.1.3(d) |
| Physical goods/services | Consumed outside app (Uber, Amazon, food delivery) | §3.1.3(e) |
| Free companion apps | App is free, companion to paid web tool, no purchase CTA | §3.1.3(f) |
| Advertising campaign purchases | Buying ad campaigns across other media types | §3.1.3 |

**B2B SaaS edge case:**  
If a user signs up on your website and logs in to the iOS app to access their subscription — this is often allowed as "multiplatform service" OR "free companion app." BUT: the iOS app cannot include a button/link that directs to your pricing page or checkout. Reviewer message pattern: *"Your app contains a call-to-action directing users to an external purchase mechanism."*

**StoreKit 2 implementation pattern (recommended):**
```swift
import StoreKit

@MainActor
class StoreManager: ObservableObject {
    @Published var products: [Product] = []
    
    func loadProducts(ids: [String]) async {
        do {
            products = try await Product.products(for: Set(ids))
        } catch {
            // Handle gracefully — don't show broken paywall
            print("Failed to load products: \(error)")
        }
    }
    
    func purchase(_ product: Product) async throws -> Transaction? {
        let result = try await product.purchase()
        switch result {
        case .success(let verification):
            let transaction = try checkVerified(verification)
            await transaction.finish()
            return transaction
        case .pending:
            return nil
        case .userCancelled:
            return nil
        @unknown default:
            return nil
        }
    }
    
    func checkVerified<T>(_ result: VerificationResult<T>) throws -> T {
        switch result {
        case .unverified:
            throw StoreError.failedVerification
        case .verified(let safe):
            return safe
        }
    }
}
```

**External payment links — by storefront:**  
- US storefront: External links/CTAs allowed without entitlement (current guidelines, Feb 2026)
- EU: Requires `StoreKitExternalPurchaseLinkEntitlement`; must use disclosure sheet
- Netherlands dating apps: Separate entitlement program
- South Korea: Separate entitlement; separate binary required

**Community sources:**  
- RevenueCat: https://www.revenuecat.com/blog/growth/the-ultimate-guide-to-app-store-rejections/  
- StackOverflow: https://stackoverflow.com/questions/tagged/in-app-purchase+rejected  
- Reddit r/iOSProgramming: search "3.1.1 rejected"

---

### #4 — Guideline 3.1.2: Subscription Paywall Design (Business Section)

**Apple anchor:** https://developer.apple.com/app-store/review/guidelines/#subscriptions  
**Apple HIG:** https://developer.apple.com/design/human-interface-guidelines/in-app-purchase  
**Probability: 3/5 | Fix impact: 4/5**

#### Required subscription paywall elements

```
✅ Price and billing period: prominent, legible, correct currency
✅ Auto-renewal disclosure: "Subscription auto-renews unless canceled"  
✅ Trial terms: exact duration + what happens after (not just "free trial")
✅ Restore Purchases button
✅ Link to Terms of Use (can be in App Store description for metadata)
✅ Link to Privacy Policy
✅ Cancel instructions or "Manage subscriptions" link
```

**Common rejection:** Trial price emphasized over billing price  
Bad paywall: Large "$0 FREE TRIAL" with tiny "$9.99/month after" in gray text  
Good paywall: Equal visual weight for trial period and billing amount

**Apple's stronger wording that often gets missed:** the **amount that will be billed must be the most prominent pricing element**. For annual plans, the annual billed amount should dominate the layout; any monthly equivalent or savings callout must be subordinate in size and emphasis.

```swift
// Subscription paywall disclosure text — compliant template
var disclosureText: String {
    """
    \(trialDuration)-day free trial, then \(price)/\(period). \
    Subscription auto-renews unless canceled at least 24 hours before the end \
    of the current period. Manage in App Store Settings.
    """
}
```

**Issues fetching subscription products during review:**
- Symptom: Reviewer sees empty paywall or "Loading..." spinner
- Cause 1: IAP not submitted with binary → resubmit with IAPs
- Cause 2: Apple sandbox outage → resubmit same build; mention in Resolution Center
- Cause 3: Products not approved → check ASC IAP status

**Quality-of-life requirement that reduces rejection risk:** Apple recommends giving subscribers an easy way to view their current status and manage/cancel subscriptions in-app. The system-provided management UI can be surfaced with `showManageSubscriptions(in:)`.

```swift
import StoreKit

@MainActor
func openManageSubscriptions(in scene: UIWindowScene) async {
    do {
        try await AppStore.showManageSubscriptions(in: scene)
    } catch {
        print("Unable to open subscription management: \(error)")
    }
}
```

**Community sources:**  
- RevenueCat: https://www.revenuecat.com/blog/growth/the-ultimate-guide-to-app-store-rejections/  
- Adapty: https://adapty.io/blog/app-store-rejection/  

---

### #5 — Guideline 4.2: Minimum Functionality (Design Section)

**Apple anchor:** https://developer.apple.com/app-store/review/guidelines/#minimum-functionality  
**Probability: 4/5 | Fix impact: 4/5**

**Rejection message pattern:** *"Your app is similar to a website" / "Not sufficiently app-like" / "Doesn't provide enough utility beyond what Safari offers."*

#### WebView rejection triggers
- Primary content is a single WKWebView loading a URL
- No native navigation; browser-back-button behavior
- No offline capability
- No device integrations (push, camera, location, etc.)
- No native UI components beyond the web container

#### How to add "native value" to pass review

```swift
// Signal to App Review that this is a native app, not a wrapper:

// 1. Push notifications (requires real implementation, not just permission)
UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .badge, .sound]) { ... }

// 2. Offline mode with cached content
URLCache.shared = URLCache(memoryCapacity: 10_000_000, diskCapacity: 100_000_000)

// 3. Native navigation stack over web content
NavigationStack {
    WebContentView()
        .navigationTitle("Home")
        .toolbar { /* native toolbar */ }
}

// 4. Share sheet (native sharing, not web share)
ShareLink(item: URL(string: currentURL)!)

// 5. Haptic feedback on interactions
let impactFeedback = UIImpactFeedbackGenerator(style: .medium)
impactFeedback.impactOccurred()
```

**Template app exception (§4.2.6):**  
Apps made from commercial app generators must be submitted by the provider of the app's content, not the generator company. Aggregator/picker model (one app hosting many clients) is the allowed architecture.

**Community sources:**  
- OneMobile: https://onemobile.ai/common-app-store-rejections-and-how-to-avoid-them/  
- decode.agency: https://decode.agency/article/app-store-rejection/  
- Medium: search "App Store 4.2 rejection webview fix"

---

### #6 — Guideline 5.1.1(v): Account Deletion & Login Gating (Legal Section)

**Apple doc:** https://developer.apple.com/support/offering-account-deletion-in-your-app/  
**Probability: 3/5 | Fix impact: 3/5**

**Two separate rules:**

1. **Mandatory login rule**: If an app requires account creation for features that aren't account-dependent, the reviewer will reject it.
   - Rejected: Lock entire app behind login when core features don't need identity
   - Allowed: Lock personalized/social/sync features behind login; let users browse without it

2. **Account deletion rule**: If account creation is supported, in-app deletion is mandatory.
   - Must delete account AND data (with legal exception disclosure)
   - Can't redirect to web page or email support@yourapp.com for deletion

**Reviewer message patterns:**  
- *"App requires users to register before they can access features that are not account-based"*
- *"Account deletion feature was not found. Offering users the option to temporarily deactivate or disable their account is not enough."*

---

### #7 — Guideline 1.2: User-Generated Content (Safety Section)

**Apple anchor:** https://developer.apple.com/app-store/review/guidelines/#user-generated-content  
**Probability: 3/5 | Fix impact: 5/5**

**Four required components (all mandatory):**
1. Filtering mechanism for objectionable material
2. Reporting mechanism with timely responses
3. Ability to block abusive users
4. Published contact information

**What immediately disqualifies an app:**
- Random/anonymous chat (Chatroulette-style) → not allowed on App Store
- App primarily used for pornographic content
- "Hot-or-not" voting on real people
- Bullying or physical threats facilitated by the app

**NSFW content edge case:**  
If UGC can include NSFW content from a connected web service: it must be **hidden by default** and only enabled when user turns it on via **the developer's website** (not in-app settings). This is an explicit Apple allowance in the guidelines.

**Creator app requirement (§1.2(a)):**  
Creator apps must provide a way for users to flag content exceeding the app's age rating, plus age verification to restrict underage users.

**Implementation pattern:**
```swift
struct ContentReportView: View {
    let content: UserContent
    @State private var reportReason: ReportReason = .spam
    
    var body: some View {
        Form {
            Section("Why are you reporting this?") {
                Picker("Reason", selection: $reportReason) {
                    ForEach(ReportReason.allCases) { reason in
                        Text(reason.displayName).tag(reason)
                    }
                }
                .pickerStyle(.inline)
            }
            
            Button("Submit Report") {
                Task { await submitReport(content: content, reason: reportReason) }
            }
        }
        .navigationTitle("Report Content")
    }
}

// Must also implement block:
func blockUser(_ userId: String) async { ... }

// And published contact info in app:
Link("Contact Support", destination: URL(string: "mailto:support@yourapp.com")!)
```

---

### #8 — Guideline 2.3.1: Hidden Features & Accurate Metadata (Performance Section)

**Apple anchor:** https://developer.apple.com/app-store/review/guidelines/#accurate-metadata  
**Probability: 3/5 | Fix impact: 3/5**

**What Apple considers "hidden":**
- Features reachable only via undocumented gestures or deep links
- Feature flags enabled server-side after review approval
- Debug menus accessible by shake or tap sequence not shown to reviewers
- Functionality that only activates in non-review geography/time

**Reviewer message:** *"Your app may contain features or content that are not defined in your app description or Notes for Review."*

**Note for Review template for feature flags:**
```
Notes for Review — Feature Flags:

This app uses remote feature configuration. All features visible to App Review 
are currently enabled in the review environment. Feature "X" requires [condition] 
and can be accessed via Settings → Developer → Enable X. No features are gated 
on geography or time that would prevent review.
```

**Metadata accuracy checklist:**
- Screenshots match the current build's actual UI
- "What's New" lists all material changes with specificity (generic descriptions rejected)
- App description doesn't claim features the app doesn't have
- Do not claim virus scanning, iOS optimization, or other impossible capabilities

---

### #9 — Guideline 2.5.2: Code Download & Hot Updates (Performance Section)

**Apple anchor:** https://developer.apple.com/app-store/review/guidelines/#software-requirements  
**Probability: 2/5 | Fix impact: 5/5**

**What is prohibited:**
- Downloading and executing code that adds features or functionality after review
- Hot-patching frameworks: JSPatch, Rollout.io, certain React Native "code push" configurations
- Remote JavaScript bundles replacing the reviewed bundle
- `itms-services://` used for code installation

**What is allowed:**
- Updating content (images, text, JSON config, remote data feeds) without new executables
- Server-driven UI where the structure is fixed but data varies
- Educational apps where downloaded code is viewable and editable by the user (§2.5.2 exception)
- HTML5/JavaScript mini-apps under §4.7 rules (separate compliance track)
- Game emulators downloading ROMs the user owns

**React Native / Expo OTA edge case:**
- React Native Metro bundler's "dev server" connection must NOT be in release builds
- Expo Updates / CodePush: content-only updates (no new native modules) may be acceptable; functionality changes are not
- Reviewer message when rejected: *"Your app loads a JavaScript bundle from a remote server that can change the app's behavior after review."*

```swift
// Info.plist — disable remote JS bundle fetching in release
// In Expo config:
{
  "expo": {
    "updates": {
      "enabled": false  // or restrict to content-only
    }
  }
}
```

**StackOverflow fix patterns:**  
https://stackoverflow.com/questions/tagged/react-native+hot-update+app-store  
https://stackoverflow.com/questions/33859344/is-it-possible-to-update-ios-app-without-submitting-to-app-store

---

### #10 — Guideline 2.5.1: Private / Non-Public APIs (Performance Section)

**Apple anchor:** https://developer.apple.com/app-store/review/guidelines/#software-requirements  
**Probability: 2/5 | Fix impact: 4/5**

**Commonly rejected private API patterns:**

| Private API | Public alternative |
|-------------|-------------------|
| `prefs:root://` and other Settings deep links | Open **your** app's settings: `UIApplication.shared.open(URL(string: UIApplication.openSettingsURLString)!)` (SwiftUI: `openSettingsURLString`). Do not deep-link into arbitrary system panes. |
| `IOKit` private symbols | `UIDevice`, `ProcessInfo`, documented `DeviceCheck` / server attestation where appropriate |
| `_UIApplicationSystemAppearance` | `UITraitCollection.current.userInterfaceStyle` |
| Undocumented selector calls | Check Apple's public Swift/ObjC docs |
| Private SpringBoard APIs | Not replaceable — remove the feature |

**Automated scanner detection:**
- Apple runs binary analysis on submissions; private symbol usage will flag
- Third-party SDK may introduce private APIs without your knowledge

**Debug approach:**
```bash
# Private URL schemes (common rejection)
strings YourApp.app/YourApp | grep -E "prefs:root|App-Prefs"

# Linked frameworks / dylibs (audit third-party binaries)
otool -L YourApp.app/YourApp

# Xcode Organizer: Validate App (often surfaces private API / entitlement issues)
```
Note: `nm -u` on the main binary is **not** a reliable private-API detector; Apple's checks and vendor tools target undocumented selectors and linked private frameworks.

**StackOverflow case:**  
https://stackoverflow.com/questions/218560/app-rejected-3-times-clueless-how-to-fix-issue  
https://apple.stackexchange.com/questions/218560/app-rejected-3-times-clueless-how-to-fix-issue

---

### #11 — Guideline 5.6 / 3.1.2(a): Manipulative Practices, Scam Signals, and Dark Patterns

**Apple anchors:** https://developer.apple.com/app-store/review/guidelines/#legal  
https://developer.apple.com/app-store/review/guidelines/#subscriptions  
**Probability: 3/5 | Fix impact: 5/5**

This is a meaningful gap in most rejection guides: Apple can reject or later remove apps that technically "work" but undermine customer trust.

**Apple language worth treating as a blocker:**  
Apps must never:
- trick users into unwanted purchases,
- force unnecessary data sharing,
- raise prices in a tricky manner,
- charge for features/content not delivered,
- or engage in manipulative practices in or outside the app.

#### High-risk dark-patterns

| Pattern | Why Apple flags it | Safer fix |
|---------|--------------------|----------|
| Huge "Free Trial" text with tiny billed amount | Misleading billing emphasis | Make billed amount the most prominent price element |
| Countdown timers / fake scarcity on evergreen offers | Can look deceptive or scammy | Use truthful, server-backed promotion windows only |
| Hiding cancel/manage path | Prevents informed subscription choice | Add in-app entry point to system subscription management |
| Gating app use on ratings, downloads, or contact upload | Explicitly disallowed | Remove forced actions; keep requests optional |
| Selling push notifications / camera / iCloud access as premium features | Monetizing built-in OS capabilities is prohibited | Bundle premium value around your service, not OS primitives |
| Promising AI or diagnostic capabilities the app doesn't actually deliver | Misleading users / false claims | Narrow claims to what is verifiable and shown in-app |

**Related clauses developers miss:**
- §3.2.2(x): cannot force users to rate/review/download apps to unlock access
- §4.10: cannot monetize built-in capabilities
- §5.6: manipulative behavior can lead to removal from the Developer Program

**If you need a rating prompt, use Apple's API and keep it optional:**
```swift
import StoreKit

@MainActor
func requestReviewIfAppropriate(in scene: UIWindowScene) {
    SKStoreReviewController.requestReview(in: scene)
}
```

Do not tie this to rewards, unlocks, or required progression.

---

### #12 — Age Ratings, Creator Apps, and 2026 Submission Interruptions

**Apple anchors:** https://developer.apple.com/news/upcoming-requirements/?id=07242025a  
https://developer.apple.com/app-store/review/guidelines/#user-generated-content  
**Probability: 2/5 | Fix impact: 3/5**

Apple updated its age-rating system and requires developers to answer new age-rating questions in App Store Connect to avoid interruption when submitting updates.

#### Important 2026 changes
- Ratings were automatically updated for Apple platforms running the new OS family
- Developers must complete the updated age-rating questionnaire in App Store Connect
- Creator apps must let users identify content exceeding the app's age rating
- Creator apps must restrict underage access using **verified or declared age**

#### Practical implication for AI + UGC apps
- If the app can generate or expose mature text/images/video, the age-rating workflow is no longer just metadata
- The product needs a real enforcement mechanism, not only a declared App Store age rating
- Apple's Declared Age Range API is an emerging tool to support this on newer OS versions, but the policy obligation exists regardless

---

## Supply Chain: Privacy Manifests & SDK Signatures

**Policy effective:** May 1, 2024  
**Apple doc:** https://developer.apple.com/documentation/bundleresources/privacy_manifest_files  
**SDK list:** https://developer.apple.com/support/third-party-SDK-requirements/

### What's required (two tracks)

**Track A — Listed third-party SDKs**  
When you submit a **new** app or an **update that newly adds** a binary dependency from [Apple's list](https://developer.apple.com/support/third-party-SDK-requirements/), that SDK must include `PrivacyInfo.xcprivacy` and (when used as a binary dependency) meet signature expectations. The list includes major mobile stacks (e.g. **Flutter**, **Firebase** pieces, **Facebook SDKs**, **Realm**, **RxSwift**, **Lottie**, **SDWebImage**, **UnityFramework**, etc.) — always re-read the live page before submission.

**Track B — Required reason APIs (your code or any executable in the app)**  
Per [Describing use of required reason API](https://developer.apple.com/documentation/bundleresources/privacy_manifest_files/describing_use_of_required_reason_api): if code uses covered API categories (file timestamp, disk space, UserDefaults, system boot time, active keyboard, and other listed categories), the **bundle that contains that executable** must declare `NSPrivacyAccessedAPITypes` with **approved reason codes**. Effective **May 1, 2024**, uploads without this can be rejected. **Fingerprinting is never allowed** even with user permission. Third-party SDKs must declare their **own** use; your app cannot "cover" an SDK's required-reason usage via the app-level manifest alone.

### ITMS / email patterns (non-exhaustive)

| Signal | Typical cause | First fix |
|--------|----------------|-----------|
| `ITMS-91061` | Listed SDK missing `PrivacyInfo.xcprivacy` | Upgrade/replace SDK; see [Apple Developer Forums](https://developer.apple.com/forums/tags/privacy-manifest) |
| Upload rejected / email about required-reason APIs | App or embedded binary uses covered APIs without `NSPrivacyAccessedAPITypes` | Add reasons to the correct target's manifest per Apple doc |
| Invalid signature on framework | Binary SDK from unknown or tampered source | Re-download from vendor; verify Xcode signature validation |

### Required-reason API categories (examples — see Apple doc for full list)
- File timestamp, disk space — `NSPrivacyAccessedAPICategoryFileTimestamp`, `NSPrivacyAccessedAPICategoryDiskSpace`
- User defaults — `NSPrivacyAccessedAPICategoryUserDefaults`
- System boot time — `NSPrivacyAccessedAPICategorySystemBootTime`
- Active keyboard — `NSPrivacyAccessedAPICategoryActiveKeyboards`

### App-level manifest vs. App Store Connect privacy answers

The Xcode **privacy report** merges SDK manifests; it does not replace the **App Privacy** questionnaire in App Store Connect. Declarations in `PrivacyInfo.xcprivacy` (e.g. `NSPrivacyTracking`, tracking domains, collected data types) must **match** what you ship and what users see on the product page — mismatches are a common source of privacy rejections and review delays.

### ITMS-91061 fix workflow

```
1. Run: Xcode → File → Generate Privacy Report
   → Identifies which frameworks are missing manifests

2. Check SDK version notes for manifest inclusion
   → Firebase >= 10.x, Amplitude >= 8.x, Braze >= 7.x include manifests
   → If your SDK version doesn't include one, update

3. If update not available:
   → Contact SDK vendor
   → Or replace SDK
   → Last resort: manually add .xcprivacy to framework bundle during archive phase
     (Xcode Run Script Build Phase after "Copy Bundle Resources")

4. Validate before submission:
   → Products → Archive → Validate App
   → "Validate" step will surface manifest errors before upload
```

### Common SDKs and manifest status (verify with vendor for current version)
- Firebase/GoogleAnalytics: ✅ manifests in recent versions
- Facebook SDK: ✅ recent versions include manifest  
- Amplitude: ✅ from v8.x
- Mixpanel: ✅ recent versions
- Braze: ✅ from v7.x
- Adjust: ✅ recent versions

**Community discussion:**  
Reddit r/iOSDev: search "ITMS-91061 privacy manifest"  
StackOverflow: https://stackoverflow.com/questions/tagged/privacy-manifest

---

## Reviewer-Only Failure Repro (Operational Pattern)

A recurring real-world pattern is "works for all testers, fails for App Review" (often login or purchase flow).  
Community examples repeatedly recommend shipping temporary, non-PII diagnostics to capture what reviewers actually hit.

**Operational packet that resolves fastest:**
1. Add server-side request correlation for review build only (`build`, `timestamp`, `endpoint`, `status`).
2. Log auth failures with coarse reason (`invalid_credentials`, `token_expired`, `region_blocked`, `backend_unreachable`).
3. Attach reviewer-path steps + screenshots/video in Resolution Center.
4. Reply with exact clause and exact fix; avoid generic "please re-review" responses.

**Apple workflow references:**  
- Resolve unresolved issues in App Store Connect and communicate with attachments until resubmission.  
- Metadata-only rejection can be corrected and resubmitted without a new binary.

**Field evidence example:**  
- Apple StackExchange login issue thread: https://apple.stackexchange.com/questions/218560/app-rejected-3-times-clueless-how-to-fix-issue

---

## Login Services (§4.8): Sign in with Apple

**When Sign in with Apple is REQUIRED:**  
If your app offers any third-party or social login (Google, Facebook, Twitter, LinkedIn, WeChat, etc.) as the primary account setup method, you MUST also offer Sign in with Apple as an equivalent option.

**Exceptions:**
- App uses only your own login system
- Alternative app marketplace app
- Education/enterprise apps using institutional accounts
- Apps requiring government ID verification
- Client apps where users sign in directly to a specific third-party service (e.g., Gmail client)

**Reviewer message:** *"Your app uses a third-party login service but does not offer Sign in with Apple as an equivalent option."*

```swift
import AuthenticationServices

// Correct Sign in with Apple implementation
SignInWithAppleButton(.signIn) { request in
    request.requestedScopes = [.fullName, .email]
} onCompletion: { result in
    switch result {
    case .success(let authorization):
        handleAuthorization(authorization)
    case .failure(let error):
        handleError(error)
    }
}
.signInWithAppleButtonStyle(.black)
.frame(height: 50)
```

**Reference:** https://developer.apple.com/app-store/review/guidelines/#login-services  
**Implementation guide:** https://developer.apple.com/documentation/authenticationservices/implementing_user_authentication_with_sign_in_with_apple

---

## Low-Visibility but Enforced Rules (Often Missed)

### App Clips / Widgets / Siri
- App Clips cannot contain advertising.
- Widgets, extensions, and notifications must be related to app content and functionality.
- SiriKit/Shortcuts intents must match your app's real domain; registering unrelated intents is rejectable.

### Forced Ratings / Store Actions
- Apps cannot force users to rate/review the app, download other apps, or perform store-related actions to unlock functionality.
- This is often cited together with scam/manipulative pattern enforcement.

### Developer Conduct / Manipulation Risk
- Repeated manipulative behavior (misleading metadata, discovery fraud, fake reviews) can escalate from rejection to account-level action.
- Your response style in Resolution Center should be factual, respectful, and evidence-based.

---

## Kids Category (§1.3)

Especially strict — misclassification or rule violations cause rapid rejection.

**Hard rules:**
- No links out of the app (without parental gate)
- No third-party advertising
- No third-party analytics (limited exceptions if zero PII collected)
- No in-app purchases without parental gate
- No IDFA / tracking identifiers
- Must comply with COPPA (US), GDPR-K (EU), and applicable children's privacy laws

**Age rating trap:**  
Using "For Kids" or "For Children" in metadata requires Kids Category. If you use those terms in non-Kids Category apps, Apple will require removal.

**Reference:** https://developer.apple.com/app-store/review/guidelines/#kids-category

---

## VPN Apps (§5.4)

- Must use `NEVPNManager` API only
- Must be submitted by an organization (not individual developer)
- Cannot sell or share user data to third parties
- Must pre-disclose data collection before purchase
- Must have necessary regional licenses where VPN operation requires them
- Non-compliance → removal from App Store + potentially from Developer Program

---

## Health & Medical Apps (§5.1.3, §1.4.1)

**High-scrutiny category:**
- Cannot claim to measure blood pressure, blood oxygen, or X-ray from device sensors alone
- Cannot write false/inaccurate data to HealthKit
- Cannot store personal health information in iCloud
- Cannot use health data from HealthKit for advertising/data mining
- Apps claiming diagnostic or treatment capabilities face stricter review + may need regulatory clearance

**Regulated domain rule:**  
Medical apps making clinical claims should be submitted by the legal entity providing the service, not an individual.

---

## Gaming, Gambling, and Lotteries (§5.3)

**Apple anchor:** https://developer.apple.com/app-store/review/guidelines/#legal (scroll to **5.3**)

- Real money gambling requires licensing in every jurisdiction where distributed
- Apps must be geo-restricted to licensed jurisdictions
- Free on App Store (gambling apps cannot charge download fee)
- `in-app purchase` cannot be used to buy gambling credits
- Loot boxes: must disclose odds of each item type before purchase (§3.1.1)
- Binary options trading apps: not permitted
- Cryptocurrency futures/ICOs: only from licensed financial institutions

### Rejection / review patterns (gambling & contests)

| Pattern | Typical issue | Fix |
|---------|----------------|-----|
| Real-money play without license proof | Reviewer cannot verify legality in reviewer’s storefront | Geo-restrict; attach licenses in Notes for Review; expect longer review |
| Paid app + gambling | **§5.3** — gambling apps must be **free** | Change pricing model |
| IAP buys chips | IAP must not purchase **gambling credit** | Use compliant model per guideline |
| Sweepstakes / raffle | Apple shown as sponsor | Rules must state **Apple is not a sponsor** (**§5.3.2**) |
| Loot box odds hidden | **§3.1.1** | Show odds **before** purchase |

**Skill cross-reference:** [SKILL.md](SKILL.md) **Gate K** · [EDGE-CASES — Gambling gray zone](EDGE-CASES.md#gambling-gray-zone)

---

## Safety, violence, weapons & speech (§1.1)

**Apple anchor:** https://developer.apple.com/app-store/review/guidelines/#safety

| Pattern | Guideline hook | Mitigation |
|---------|----------------|------------|
| Gratuitous gore / harm | **§1.1.2** realistic violence | Tone down; contextualize; match age rating; games avoid solely targeting real groups |
| Weapons commerce / reckless use | **§1.1.3** | Remove purchase facilitation or illegal encouragement; legal review for region |
| Hate / harassment disguised as “debate” | **§1.1.1** | Moderation, reporting, clear rules; satire must be genuine professional satire per guideline |
| “Free speech” appeal | N/A — platform rules | Apple applies **§1.1** / **§1.2**; not a public-forum legal defense in review |
| Terrorism / CSAM / serious harm | **§1.1** + law | Do not ship; legal + safety response |

**Skill cross-reference:** [SKILL.md](SKILL.md) **Gate K** · [EDGE-CASES — Political and satirical content](EDGE-CASES.md#political-and-satirical-content-111)

---

## Sandboxing & privileged system access (iOS + macOS)

| Surface | Rule of thumb | Apple pointer |
|---------|----------------|---------------|
| iOS data isolation | Stay in **sandbox** / **documented** sharing APIs; no peeking into other apps’ containers | **§2.5.1**, **ADPLA** Program Requirements |
| Mac App Store | **Sandbox**; single bundle; updates via MAS only | **§2.4.5** — [REFERENCE — Mac App Store](#mac-app-store-additional-rules-245) |
| VPN | **Organization** developer; `NEVPNManager`; no selling user data | **§5.4** |
| MDM | Commercial / edu / gov; Apple-granted capability; disclosures | **§5.5** |
| Private Settings URLs | `prefs:root` / `App-Prefs` → rejection risk | **§2.5.1** — use `openSettingsURLString` |
| macOS Automation / Accessibility | Use for **real** feature; avoid “remote control” of other apps for non-a11y automation | Document in Notes for Review; match entitlements |

**Skill cross-reference:** [SKILL.md](SKILL.md) **Gate K**, **Gate F**

---

## Cryptocurrency & NFTs (§3.1.5)

- Wallets: org enrollment required (not individual)
- Mining: only off-device (cloud-based)
- Exchanges: requires proper licensing by jurisdiction
- NFTs: cannot unlock app features or functionality
- NFT browsing allowed; external purchase links subject to storefront rules

---

## §5.2 IP, Apple marks, §5.1.5 location, ads & push (rejection patterns)

**Apple anchors:**  
- Legal hub: https://developer.apple.com/app-store/review/guidelines/#legal  
- Privacy hub: https://developer.apple.com/app-store/review/guidelines/#privacy  

**Skill cross-reference:** [SKILL.md](SKILL.md) — **Gate C** (location, ATT, ads, push), **Gate E** (IP & marks), **Gate K** (safety, gambling, sandbox), **Gate J** (ADPLA), **Gate A** (encryption/export).

### §5.2 Intellectual Property

**Guideline gist:** *"Make sure your app only includes content that you created or that you have a license to use. Your app may be removed if you've stepped over the line and used content without permission."*

| Sub-cause | Reviewer / risk pattern | Fix |
|-----------|-------------------------|-----|
| Unlicensed assets | Metadata or binary contains **trademark**, **character**, **music**, or **stock** content you cannot document rights for | Replace with owned/licensed assets; keep license trail for review disputes |
| Misleading affiliation | App name, icon, or screenshots suggest **official** tie to brand X | Rename/redesign; remove third-party logos from screenshots unless permitted |
| Third-party scraping / rip | **§5.2.3** — saving, converting, or downloading media from services (e.g. music/video platforms) without permission | Remove download paths; integrate only via official APIs/SDK terms |
| Terms violation | **§5.2.2** — app uses or monetizes a third-party API/content against their ToS | Re-read partner agreements; remove features or obtain written permission |
| Copycat (related) | **§5.2.1** + **§4.1** — "copycat" or misleading representation of another *developer's* app | Differentiate icon/name/UI; see **Gate E** in [SKILL.md](SKILL.md) |

**IP dispute (another party claims infringement):** Apple’s form — https://www.apple.com/legal/intellectual-property/dispute-forms/index.html  

### §5.2.4 Apple endorsements & §5.2.5 Apple-like UI

**§5.2.4 quote (abridged):** *"Don't suggest or imply that Apple is a source or supplier of the App, or that Apple endorses any particular representation regarding quality or functionality."* Editor’s Choice and similar badges are **applied by Apple only**.

**§5.2.5 gist:** Do not create an app **confusingly similar** to Apple’s products, **App Store / iTunes / Messages**-like interfaces, or Apple **advertising** themes.

| Sub-cause | Pattern | Fix |
|-----------|---------|-----|
| Fake “Featured by Apple” | Marketing, screenshots, or landing page show **unauthorized** Apple badges or rankings | Use only [Marketing Resources](https://developer.apple.com/app-store/marketing/guidelines/) assets where allowed; remove fake badges |
| App Store clone UI | Onboarding mimics **App Store** purchase sheets or system **Settings** to trick users | Redesign; don’t mimic first-party store/system chrome |
| Notarization overclaim (macOS) | Site says Apple **“approved”** or **“verified security”** because of notarization | Reword per ADPLA — notarization ≠ Apple endorsement of quality (**Gate J** in [SKILL.md](SKILL.md)) |

**Apple trademark / marketing rules (contract + guidelines):** https://www.apple.com/legal/intellectual-property/guidelinesfor3rdparties.html  

### §5.1.5 Location Services

**Guideline gist:** *"Use Location Services in your app only when it is directly relevant to the features and services provided by the app."* Notify and obtain consent; explain purpose; location APIs should not be used for **emergency services** or **autonomous control** of vehicles/aircraft (with narrow exceptions in the guideline text).

| Sub-cause | Pattern | Fix |
|-----------|---------|-----|
| Irrelevant location | App requests location for **ads/analytics** only while claiming “local features” | Narrow collection; disclose in privacy label and policy; align purpose strings |
| Misleading purpose string | **§5.1** — string says X but app uses location for Y | Rewrite `NSLocation*` keys to match actual use |
| Broken / custom consent UI | Custom pre-alert **blocks** or **spoofs** system permission (ADPLA §3.3.3(F)(iv)) | Remove; use system APIs only; accurate `purpose` copy |
| “Always on” without justification | Background location with weak user benefit | Add clear in-app rationale; trim to “When In Use” if possible |

**HIG:** https://developer.apple.com/design/human-interface-guidelines/patterns/accessing-private-data/  

### ATT, IDFA, ads, APNs (program + guidelines)

| Sub-cause | Pattern | Fix |
|-----------|---------|-----|
| Tracking without ATT | Ads/analytics fingerprint users cross-app without **App Tracking Transparency** when required | Integrate ATT; show prompt before accessing IDFA for tracking |
| Feature-gated tracking | Core functionality requires “Allow Tracking” | Remove gate; degrade gracefully ([ATT docs](https://developer.apple.com/documentation/apptrackingtransparency)) |
| IDFA misuse (contract) | Using IDFA for **non-ad** purposes, or **re-linking** after user resets advertising identifier (ADPLA §3.3.3(E)) | Restrict IDFA to advertising; honor reset |
| Promotional push without opt-in | **APNs** used for upsell/marketing without documented user consent path (Attachment 1) | Add explicit opt-in copy + in-app opt-out; Pass-only promo exception for Pass-related use |
| Wi‑Fi / VPN data for ads | **Network Extension** or Wi‑Fi info used to **profile** or infer location when user disabled location (ADPLA §3.3.3(G)) | Scope networking entitlements to stated VPN/filtering features only |

**Kids (§1.3):** Apps **primarily for kids** must not include **third-party analytics** or **third-party advertising** — frequent rejection if SDKs slip in.

### Website / Safari push notifications (ADPLA Attachment 1 §3)

| Sub-cause | Pattern | Fix |
|-----------|---------|-----|
| Misleading sender | Push appears to come from **another** brand or site | Send only under **your** registered site brand; include **your** icon/logo |
| Third-party mark in payload | Competitor or partner trademark without license | Obtain rights or remove mark |
| Apple promo use of your creative | Apple may use **screenshots** of your Safari pushes and **your** marks in Apple marketing unless you **exclude in writing** the portions you cannot grant | Identify restricted assets to Apple if needed |

---

## Export Compliance

Blocks **upload** or **distribution** until resolved — often surfaced in App Store Connect, not always as a “guideline” paragraph in Resolution Center.

**Info.plist keys to set:**
```xml
<!-- If using only standard HTTPS (ATS) and no other encryption: -->
<key>ITSAppUsesNonExemptEncryption</key>
<false/>

<!-- If using custom/additional encryption: -->
<key>ITSAppUsesNonExemptEncryption</key>
<true/>
<key>ITSEncryptionExportComplianceCode</key>
<string><!-- UUID from your ERN approval --></string>
```

**Operational patterns:**
- Answer **encryption questions** in App Store Connect truthfully for **each** build; mismatches between binary capabilities and answers can **hold** processing.
- Apps with **non-exempt** encryption may need U.S. **EAR** / **BIS** self-classification, **CCATS**, or other filings before you can ship — legal/compliance owns this, not only engineering.
- **ADPLA** upload certifications (e.g. Schedule 1 delivery, **§5.3** notarization) assume you are not uploading **ITAR**/restricted payloads without authorization.
- **macOS notarization**: export rules apply to what you upload to Apple’s notary service; keep plist + documentation aligned.

**References:**  
- https://developer.apple.com/documentation/security/complying_with_encryption_export_regulations  
- https://developer.apple.com/help/app-store-connect/reference/app-information/export-compliance-documentation-for-encryption  

---

## IPv6-Only Networks (§2.5)

Apple reviews on IPv6-only networks. Apps that use hardcoded IPv4 addresses or don't support IPv6 will fail.

**Test locally:**
- Mac: System Preferences → Sharing → Internet Sharing (creates IPv6-only NAT64 hotspot)
- Xcode: Scheme → Run → Network Link Conditioner

**Common issues:**
```swift
// ❌ Hardcoded IPv4
let serverURL = URL(string: "http://192.168.1.1/api")

// ✅ Use hostname (DNS resolves to IPv6)
let serverURL = URL(string: "https://api.yourservice.com/api")

// ❌ Checking for "0.0.0.0" or IPv4-style checks
// ✅ Use Network framework for connectivity instead of Reachability with IPv4 assumptions
```

**StackOverflow:** https://stackoverflow.com/questions/tagged/ipv6+ios+app-store

---

## WKWebView Requirement

If your app includes a browser or web content viewer, it must use `WKWebView` (not `UIWebView`, which is deprecated and rejected).

```swift
// ❌ Rejected — deprecated
import UIKit
let webView = UIWebView()

// ✅ Required
import WebKit
let webView = WKWebView()
```

---

## Mac App Store — additional rules (§2.4.5)

iOS guidance applies to many topics, but **Mac App Store** submissions have extra hard rules.

- **Sandbox** — Appropriate sandboxing; follow macOS file-system expectations.
- **Xcode packaging only** — **No third-party installers**; use technologies provided in Xcode.
- **Self-contained bundle** — Single installation bundle; **no** installing code or resources into shared locations to add functionality.
- **No silent persistence** — No auto-launch at login without consent; no background processes after quit without consent; no auto-adding Dock icons or desktop shortcuts.
- **No downloading** standalone apps, kexts, or extra code to materially change the app after review.
- **No root / setuid** — No privilege escalation.
- **No license screen at launch**, **no license keys**, **no custom copy protection** (the store handles distribution).
- **Updates** — Must ship through the **Mac App Store** only.
- **Shipping OS** — Run on currently shipping macOS; avoid deprecated optional stacks (guidelines cite e.g. Java).
- **Languages** — All localizations in **one** bundle.

**Sparkle** and other non-MAS auto-updaters are for **direct distribution**, not Mac App Store apps.

---

## CallKit, VoIP, and caller identity

Apps using **CallKit** for VoIP or call management face **telecom-level** review: caller ID and metadata must be accurate and not deceptive. Follow [CallKit](https://developer.apple.com/documentation/callkit) documentation. PushKit/VoIP without legitimate call flows has historically triggered **misuse of background modes** rejections.

---

## EU Digital Services Act — trader status (App Store Connect)

Can **block or delay** submission until completed (not always framed as a "guideline" rejection):

- **Articles 30–31 DSA** — Apple verifies and displays trader **address, phone, and email** on EU product pages.
- **Trader self-assessment** — Required; hobbyists may declare non-trader where accurate.
- **Per-app** trader toggle under **App Information → Digital Services Act**.
- Optional **Labels and Markings URL** for EU product markings.

[Manage EU DSA trader requirements](https://developer.apple.com/help/app-store-connect/manage-compliance-information/manage-european-union-digital-services-act-trader-requirements/)

---

## Accessibility & HIG (soft drivers of 2.1 / 4.2)

Apple may not cite a single "accessibility guideline," but **broken layouts**, **unlabeled controls** under VoiceOver, and **Dynamic Type** failures make review flows fail or feel "unfinished." Run VoiceOver on primary paths and check [Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/).

---

## Community Case Study Index

| Rejection | Platform | Thread |
|-----------|----------|--------|
| Hot update (React Native) rejected under 2.5.2 | StackOverflow | https://stackoverflow.com/questions/33859344 |
| Private API (IOKit) flagged by scanner | StackOverflow | https://apple.stackexchange.com/questions/218560 |
| IAP "can't locate purchase" loop | Reddit | r/iOSProgramming search "2.1(b) IAP" |
| Social app rejected under 1.2 (no report/block) | Reddit | r/iOSDev search "1.2 UGC rejection" |
| B2B app with Stripe link rejected under 3.1.1 | Medium | search "iOS App Store rejection Stripe IAP" |
| Privacy policy URL 404 at submission | Blog | https://decode.agency/article/app-store-rejection/ |
| Subscription paywall price not prominent | RevenueCat | https://www.revenuecat.com/blog/growth/the-ultimate-guide-to-app-store-rejections/ |
| ITMS-91061 privacy manifest | Reddit | r/iOSDev search "ITMS-91061" |
| Account deletion missing | dev.to | https://dev.to/xurxodev/how-i-got-a-rejected-app-approved-in-the-ios-app-store-gg0 |
| WebView 4.2 rejection | uxcam | https://uxcam.com/blog/app-store-rejection-reasons/ |
| Mandatory login for non-account features | Adapty | https://adapty.io/blog/app-store-rejection/ |
| Missing Sign in with Apple | Quora | https://www.quora.com/My-first-app-has-just-been-rejected-on-Apple-App-Store-what-should-I-do |
| §5.2 IP / trademark in metadata or binary | Reddit / SO | search `app store rejection` + `5.2` or `intellectual property` |
| §5.3 gambling / real-money / IAP chips | Reddit / legal | search `app store rejection gambling` `5.3` |
| §1.1 violence / weapons / objectionable | Reddit | search `1.1 rejection` `app store` |
| Location purpose string / irrelevant tracking | Reddit r/iOSDev | search `location rejection` `NSLocationUsageDescription` |
| ATT missing or tracking without consent | Apple forums / SO | search `AppTrackingTransparency rejection` |
| Export compliance / encryption upload stuck | Apple Developer forums | search `ITSAppUsesNonExemptEncryption` `export compliance` |

---

## App Review Process Levers That Reduce Rejection Loops

These are not guidelines, but they materially improve outcomes and are easy to overlook.

- **Attachments remain available until resubmission**: you can send screenshots, demo videos, and supporting documents directly in App Review messaging
- **Metadata-only rejections don't require a new binary**: fix metadata and resubmit the same build
- **Multi-item submissions are flexible**: remove rejected items and continue accepted items
- **App Review appointments exist**: Apple offers 30-minute review/process consultations over Webex for difficult cases
- **Bug-fix submission carveout**: for already-live apps, Apple may let the current bug-fix ship while you address non-legal/non-safety issues in the next submission

**Primary docs:**  
https://developer.apple.com/distribute/app-review  
https://developer.apple.com/help/app-store-connect/manage-submissions-to-app-review/reply-to-app-review-messages

---

## Key Source Links

| Source | URL | Authority |
|--------|-----|-----------|
| App Review Guidelines (Feb 2026) | https://developer.apple.com/app-store/review/guidelines/ | Primary |
| Safety (§1.x hub) | https://developer.apple.com/app-store/review/guidelines/#safety | Primary |
| App Review preparation | https://developer.apple.com/app-store/review/ | Primary |
| App Review overview | https://developer.apple.com/distribute/app-review | Primary |
| Required reason APIs | https://developer.apple.com/documentation/bundleresources/privacy_manifest_files/describing_use_of_required_reason_api | Primary |
| Privacy manifest docs | https://developer.apple.com/documentation/bundleresources/privacy_manifest_files | Primary |
| EU DSA trader (ASC) | https://developer.apple.com/help/app-store-connect/manage-compliance-information/manage-european-union-digital-services-act-trader-requirements/ | Primary |
| DMA & apps in EU | https://developer.apple.com/support/dma-and-apps-in-the-eu/ | Primary |
| Age ratings reference | https://developer.apple.com/help/app-store-connect/reference/age-ratings-values-and-definitions/ | Primary |
| Third-party SDK requirements | https://developer.apple.com/support/third-party-SDK-requirements/ | Primary |
| Account deletion guide | https://developer.apple.com/support/offering-account-deletion-in-your-app/ | Primary |
| Reply to App Review messages | https://developer.apple.com/help/app-store-connect/manage-submissions-to-app-review/reply-to-app-review-messages | Primary |
| Auto-renewable subscriptions | https://developer.apple.com/app-store/subscriptions/ | Primary |
| Age rating updates (2026) | https://developer.apple.com/news/upcoming-requirements/?id=07242025a | Primary |
| Reader apps entitlement | https://developer.apple.com/support/reader-apps/ | Primary |
| ATT documentation | https://developer.apple.com/documentation/apptrackingtransparency | Primary |
| Legal — §5.2 IP & §5.1.5 Location (hub) | https://developer.apple.com/app-store/review/guidelines/#legal | Primary |
| Apple trademarks & third-party use | https://www.apple.com/legal/intellectual-property/guidelinesfor3rdparties.html | Primary |
| App Store marketing & identity | https://developer.apple.com/app-store/marketing/guidelines/ | Primary |
| Export compliance (ASC reference) | https://developer.apple.com/help/app-store-connect/reference/app-information/export-compliance-documentation-for-encryption | Primary |
| Encryption export regulations (dev doc) | https://developer.apple.com/documentation/security/complying_with_encryption_export_regulations | Primary |
| StoreKit 2 | https://developer.apple.com/documentation/storekit | Primary |
| RevenueCat ultimate guide | https://www.revenuecat.com/blog/growth/the-ultimate-guide-to-app-store-rejections/ | Community |
| Adapty rejection guide | https://adapty.io/blog/app-store-rejection/ | Community |
| OneMobile rejections | https://onemobile.ai/common-app-store-rejections-and-how-to-avoid-them/ | Community |
| decode.agency guide | https://decode.agency/article/app-store-rejection/ | Community |
| uxcam guide | https://uxcam.com/blog/app-store-rejection-reasons/ | Community |
| neonapps guide | https://www.neonapps.co/blogs/why-do-ios-apps-get-rejected-understanding-the-apple-review | Community |
| mobiloud guide | https://www.mobiloud.com/blog/avoid-app-rejected-apple | Community |
| goodbarber guide | https://www.goodbarber.com/help/shop/apple-rejection-r100/my-app-has-been-rejected-by-apple-a134/ | Community |
| StackOverflow iOS rejection | https://stackoverflow.com/questions/tagged/app-store-rejection | Community |
| Reddit r/iOSDev | https://www.reddit.com/r/iOSDev/ | Community |
