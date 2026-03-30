# Addendum: Overlooked Rejection Vectors, Platform-Specific Rules, and Breaking Enforcement

Material discovered during second-pass audit. These are real rejection causes and submission blockers that the primary skill files did not cover.

---

## BREAKING: Vibe Coding App Crackdown (March 2026)

**Source:** https://www.macrumors.com/2026/03/30/apple-pulls-vibe-coding-app/  
**Date:** March 30, 2026  
**Guideline:** §2.5.2

Apple has escalated enforcement against "vibe coding" apps — tools that let users generate functional apps from natural language prompts using AI.

**What happened:**
- "Anything" (raised $11M, $100M valuation) was **removed entirely** from the App Store on March 26, 2026
- Updates to **Replit** and **Vibecode** were blocked starting earlier in March 2026
- Apple cited **Guideline 2.5.2** specifically: apps may not "download, install, or execute code which introduces or changes features or functionality"
- Even attempts to comply (previewing generated apps in a web browser instead of natively) were rejected

**Apple's official statement:** There are "no specific rules against vibe coding," but apps must adhere to longstanding §2.5.2 guidelines. The generated code constitutes new executable functionality introduced after review.

**What this means for developers:**
- AI-generated code that executes within your app = §2.5.2 violation
- AI-generated code the user views/edits but doesn't execute in the host app = may be OK (educational exception)
- Generating apps and publishing them as separate App Store submissions = each submission reviewed independently
- The enforcement is **active and escalating** — not theoretical

**Safe patterns:**
- AI generates code → user copies it to Xcode → user submits their own app: OK
- AI generates code → displayed in syntax-highlighted view → user can edit: OK (educational exception)
- AI generates code → app executes the generated code natively: REJECTED

**Rejected patterns:**
- App generates mini-apps/tools and runs them in a native sandbox
- App generates UI and renders it using dynamically loaded components
- App creates functional software that operates within the host app

This is the most significant recent enforcement action related to AI code generation and directly affects any developer building AI-powered app builders, no-code tools, or LLM-based coding assistants for iOS.

---

## Age Rating System Overhaul (July 2025 — Deadline January 31, 2026)

**Source:** https://developer.apple.com/news/upcoming-requirements/?id=07242025a  
**Source:** https://developer.apple.com/help/app-store-connect/reference/age-ratings

Apple completely overhauled the age rating system in July 2025. **All apps must be compliant by January 31, 2026.**

### New age categories

| Old System | New System (iOS 26+) |
|-----------|---------------------|
| 4+ | 4+ |
| 9+ | 9+ |
| 12+ | **13+** (new) |
| 17+ | **16+** (new) |
| — | **18+** (new) |

### What changed in the questionnaire

The revised questionnaire now asks about:
- **Parental controls** — Does the app have them?
- **Messaging features** — Can users message each other?
- **Health topics** — Does the app address health in ways requiring age gating?
- **Violence depictions** — More granular than before
- **Developer override** — Developers can now **manually set a stricter age rating** if their terms of service require it (e.g., COPPA compliance requiring 13+)

### Territory-specific ratings

Different regions now have distinct rating thresholds:
- Australia, Brazil, South Korea, and others may show different age ratings for the same content
- Apps must answer the questionnaire accurately; Apple maps answers to regional standards

### The age-rating catch-22

**Known issue:** A COPPA-compliant app that requires users to be 13+ may answer "No" to all adult content questions, get assigned a 4+ rating, and then be rejected for inconsistent metadata (requiring 13+ in-app but showing 4+ on the store).

**Fix:** Use the new developer override to manually set the age rating to 13+ regardless of questionnaire answers.

### §2.3.8 metadata trap

> *"Use of terms like 'For Kids' and 'For Children' in app metadata is reserved for the Kids Category."*

Using "For Kids" or "For Children" in your app name, subtitle, description, or keywords while NOT being in the Kids Category = metadata rejection. This also applies to screenshots containing child-oriented messaging.

---

## App Size Limits & Distribution Constraints

**Source:** https://developer.apple.com/help/app-store-connect/reference/app-uploads/maximum-build-file-sizes

### Maximum build sizes by platform

| Platform | Max Uncompressed App Size | Max Executable (__TEXT) | Notes |
|----------|--------------------------|------------------------|-------|
| iOS / iPadOS (iOS 9+) | 4 GB | 500 MB | — |
| macOS | 200 GB | — | — |
| tvOS | 4 GB | 500 MB | — |
| visionOS | 4 GB | 500 MB | — |
| watchOS | 75 MB | — | Total app |
| App Clips (iOS 17+) | 100 MB | — | Digital invocations only |
| App Clips (iOS 16) | 15 MB | — | — |
| App Clips (< iOS 16) | 10 MB | — | — |

### Cellular download limit

- Apps over **200 MB** require Wi-Fi to download (default user setting)
- Users can override this in Settings, but many don't
- Best practice: keep initial download under 200 MB for maximum install conversion
- Use **Background Assets** framework (iOS 16+) to download large assets after install
- Use **On Demand Resources** (ODR) for conditional content

### Practical impact on rejection

- Apple won't reject for a large app, but conversion drops significantly above 200 MB
- watchOS apps exceeding 75 MB are rejected outright
- App Clip variants exceeding size limits after app thinning are rejected

```swift
// Use Background Assets for large content (games, ML models)
import BackgroundAssets

// Register assets in your AppDelegate
func application(_ application: UIApplication,
    handleEventsForBackgroundURLSession identifier: String) { ... }
```

---

## Screenshot & Media Requirements

**Source:** https://developer.apple.com/help/app-store-connect/reference/screenshot-specifications

### Mandatory iPhone screenshot sizes

You must provide screenshots for at least one of these sizes. Apple scales down from the largest provided.

| Display | Dimensions (portrait) | Device Examples |
|---------|----------------------|-----------------|
| 6.9" | 1320 x 2868 | iPhone 15 Pro Max, 16 Pro Max |
| 6.7" | 1290 x 2796 | iPhone 14 Pro Max, 15 Plus, 16 Plus |
| 6.5" | 1284 x 2778 | iPhone 11 Pro Max, XS Max |
| 6.1" | 1179 x 2556 | iPhone 14 Pro, 15 Pro, 16 Pro |
| 5.5" | 1242 x 2208 | iPhone 8 Plus, 7 Plus (legacy) |

### Mandatory iPad screenshot sizes (if app supports iPad)

| Display | Dimensions (portrait) |
|---------|-----------------------|
| iPad Pro 12.9" | 2048 x 2732 |
| iPad Pro 11" | 1668 x 2388 |
| iPad 10.9" | 1640 x 2360 |

### Screenshot rules

- Format: PNG or JPEG, RGB color space, **no transparency**
- 1–10 screenshots per device size
- **Exact pixel dimensions are mandatory** — even 1 pixel off can fail upload
- Screenshots must reflect the **current** build's UI — outdated screenshots trigger §2.3 rejection
- Screenshots cannot contain misleading content, fake device frames from wrong device, or competitor app names

### App Previews (video)

- Up to 3 per localization
- 15–30 seconds
- Must capture actual app usage (not marketing video with actors)
- Audio is optional but must represent the app experience

### Automation recommendation

```bash
# Fastlane snapshot — auto-generate device screenshots
fastlane snapshot --devices "iPhone 16 Pro Max" "iPhone SE" "iPad Pro (12.9-inch)"
```

---

## visionOS-Specific Requirements

**Source:** https://developers.apple.com/visionos/submit

### App motion declaration (REQUIRED)

All visionOS apps **must** declare app motion information before submission. This is a **required property** in App Store Connect.

Options:
- **No high motion** — Standard spatial app
- **High motion** — App contains frequent fast/sudden camera movements, quick turns, or unusual orientations

This declaration appears on the product page to warn users sensitive to motion. Failing to declare it blocks submission.

### visionOS design requirements

- Must follow visionOS Human Interface Guidelines
- Proper use of **Shared Space** or **Full Space** layouts
- Spatial audio integration where applicable
- Gaze tracking and gesture input must work correctly
- Avoid cluttered or blocked visuals in 3D space
- Screenshots must be captured using **Reality Composer Pro Developer Capture** (not Control Center screen recordings)

### Compatible iPad/iPhone apps on visionOS

- iPad and iPhone apps can run on Vision Pro by default in "Compatible" mode
- Developers can opt out in App Store Connect
- If opted in: app runs in a flat window; review still applies
- "Designed for visionOS" requires native visionOS build and separate review

---

## Launch Screen Requirement

**Required since:** April 30, 2020  
**Source:** https://developer.apple.com/documentation/xcode/specifying-your-apps-launch-screen

All iOS apps submitted to the App Store **must** use an Xcode storyboard or Info.plist configuration to provide a launch screen. Apps without a launch screen storyboard are rejected.

### Xcode 16 known issue

The launch screen configuration UI in Xcode 16 has known bugs:
- Blue dropdown autofill button doesn't work reliably
- Fix: manually type the launch screen filename in the field
- Verify `UILaunchStoryboardName` key exists in Info.plist

```xml
<!-- Info.plist — required -->
<key>UILaunchStoryboardName</key>
<string>LaunchScreen</string>
```

### SwiftUI apps

SwiftUI apps using `@main` App protocol may not have an explicit storyboard. Ensure the Info.plist or target settings specify the launch configuration:

```swift
// In your app's target → General → App Icons and Launch Screen
// Set "Launch Screen File" to your LaunchScreen.storyboard name
```

---

## App Transport Security (ATS) Exceptions

**Source:** https://developer.apple.com/documentation/bundleresources/information-property-list/nsapptransportsecurity

### What triggers review scrutiny

Setting `NSAllowsArbitraryLoads = YES` globally disables ATS and requires **documented technical justification** during App Review. Apps have been rejected for weak or missing justification.

### Recommended approach

Use domain-specific exceptions instead of global disabling:

```xml
<key>NSAppTransportSecurity</key>
<dict>
    <!-- ❌ Global disable — requires justification, may be rejected -->
    <!-- <key>NSAllowsArbitraryLoads</key><true/> -->
    
    <!-- ✅ Domain-specific exception — targeted and more likely approved -->
    <key>NSExceptionDomains</key>
    <dict>
        <key>legacy-api.example.com</key>
        <dict>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <true/>
            <key>NSExceptionMinimumTLSVersion</key>
            <string>TLSv1.2</string>
        </dict>
    </dict>
</dict>
```

### iOS 17+ changes

iOS 17 introduced stricter IP-based connection restrictions. Apps connecting to IP addresses (not hostnames) now require explicit `NSExceptionDomains` entries or `NSAllowsLocalNetworking` for local network access.

### ATS and web content

- `NSAllowsArbitraryLoadsInWebContent = YES` allows WKWebView to load arbitrary URLs without affecting the rest of the app's ATS policy
- This is the recommended approach for apps that display third-party web content

---

## ITMS Error Codes Beyond ITMS-91061

### ITMS-9000: Invalid Binary (catch-all)

Most common causes:
- Incomplete or missing metadata/assets
- Entitlements in app don't match App Store Connect capabilities
- Mismatched or expired provisioning profiles
- Incorrect Xcode build settings or SDK version

### ITMS-90161: Invalid Provisioning Profile

Causes:
- Expired provisioning profile
- Bundle identifier mismatch between Xcode project and profile
- Revoked distribution certificate
- App ID capabilities don't include entitlements claimed in the binary

Fix workflow:
```
1. Apple Developer Portal → Profiles → find your distribution profile
2. Verify: not expired, correct bundle ID, correct certificate
3. If expired/revoked: regenerate profile with valid certificate
4. Xcode → Signing & Capabilities → match profile
5. Clean build folder (Cmd+Shift+K) → rebuild → re-archive
```

### ITMS-90809: Deprecated API Usage

Triggered when the binary contains references to deprecated APIs like `UIWebView`. Even third-party framework binaries containing `UIWebView` symbols will trigger this. Fix: update all dependencies.

### ITMS-91053: Missing Privacy Manifest for Required Reason API

Similar to ITMS-91061 but specifically for your own code (not SDKs) using required-reason APIs without declaring the reason in your app-level `.xcprivacy` manifest.

Fix:
```xml
<!-- YourApp/PrivacyInfo.xcprivacy -->
<dict>
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryFileTimestamp</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>C617.1</string>
            </array>
        </dict>
    </array>
</dict>
```

### Entitlements mismatch pattern

```
ITMS error: "Invalid Entitlements — the entitlements in your app bundle 
do not match the ones in your provisioning profile"
```

Common when:
- You add a capability in Xcode (e.g., Push Notifications, Sign in with Apple) but don't regenerate the provisioning profile
- You remove a capability from ASC but the entitlements file still claims it
- Your development entitlements leak into the distribution build

Debug:
```bash
# Compare app entitlements vs provisioning profile
codesign -d --entitlements :- YourApp.app

# Extract provisioning profile entitlements
security cms -D -i embedded.mobileprovision
```

---

## App Store Improvements: Inactive App Removal

**Source:** https://developer.apple.com/support/app-store-improvements/

Apple proactively removes apps that meet ALL of these criteria:
1. Not updated within the **last 3 years**
2. Fail to meet a **minimum download threshold** (extremely few or zero downloads in a rolling 12-month period)

### Process
- Developer receives email notification identifying the app
- **90-day window** to submit an update
- Apps that **crash on launch** are removed immediately (no 90-day grace period)
- Removed apps can be resubmitted anytime if developer maintains active membership
- Existing users retain access; only new downloads are blocked

### Implications for long-lived apps
- Even "finished" apps need periodic updates (at minimum: rebuild with latest Xcode/SDK)
- Test old apps on latest OS versions before they break
- Keep your Developer Program membership active

---

## Regulated Medical Device Apps (2027 Deadline)

**Source:** https://www.world-today-news.com/app-store-requirements-for-regulated-medical-device-apps-in-eea-uk-and-us/

Apps in the **Health & Fitness** or **Medical** categories targeting EEA, UK, and US markets will be required to declare **"Regulated Medical Device"** status in App Store Connect metadata.

- Deadline: early 2027
- Non-compliant apps will face update rejection
- CI/CD pipelines that auto-submit may break if the metadata declaration isn't added

### What this means for health apps

If your app makes medical claims, measures health metrics, or claims to diagnose conditions:
- Start declaring medical device status now
- Ensure regulatory clearance documentation is ready
- Include FDA/CE clearance information in Notes for Review

---

## Code Signing & Provisioning: Complete Troubleshooting

### The signing chain

```
Apple Root CA
  └── Your Distribution Certificate (WWDR-signed)
       └── Provisioning Profile (cert + App ID + capabilities + devices)
            └── Your App Binary (codesigned with the cert referenced in the profile)
```

All four must be valid and internally consistent at upload time.

### Common failures and fixes

| Symptom | Cause | Fix |
|---------|-------|-----|
| "No matching provisioning profile" | Profile doesn't include current entitlements | Regenerate profile in Developer Portal with current capabilities |
| "Certificate has been revoked" | Someone on team revoked the cert | Create new distribution cert; regenerate all profiles |
| ITMS upload fails silently | Expired profile | Check profile expiry in Keychain Access |
| "Entitlements not present in provisioning profile" | Capability added in Xcode but not in App ID | Enable capability in Developer Portal → App IDs → Your App → Capabilities |
| Different signing on different Macs | Each dev has different cert/profile | Use Xcode automatic signing for development; manual for distribution with shared profile |

### Xcode automatic signing gotcha

Automatic signing generates development profiles automatically but does **NOT** handle distribution profiles. For App Store submission:
- Use manual signing for Release configuration
- Or use `fastlane match` for team-shared certificates and profiles

---

## TestFlight Review vs. App Store Review

TestFlight builds also undergo review. Key differences:

| Aspect | TestFlight Beta Review | App Store Review |
|--------|----------------------|------------------|
| Scope | Lighter; checks for crashes, basic compliance | Full guideline review |
| Timeframe | Usually < 24 hours | 24-72 hours (can be longer) |
| Demo account required? | Sometimes | Yes, for account-based apps |
| Rejection consequence | Cannot distribute to external testers | Cannot distribute to public |
| Privacy label required? | Not fully | Yes |
| IAP review? | No (sandbox only) | Yes (products must be submitted) |

### TestFlight-specific rejection triggers
- App crashes on launch (auto-rejected, no review needed)
- Export compliance not answered
- Privacy-impacting SDKs without manifests (ITMS errors at upload)
- Beta Entitlement issues

### TestFlight to App Store transition trap

Apps that work fine on TestFlight may still fail App Store review because:
- TestFlight uses sandbox receipts; App Store review uses a different environment
- IAP products must be explicitly submitted with the App Store build (TestFlight doesn't check this)
- Privacy nutrition labels are enforced for App Store but not TestFlight
- Full guideline review applies only at App Store submission

---

## Universal Links / Associated Domains

Common rejection-adjacent issue: Universal Links don't work during review because the Apple App Site Association (AASA) file is misconfigured.

### Requirements

```json
// https://yourdomain.com/.well-known/apple-app-site-association
// Must be served with Content-Type: application/json
// Must be accessible without redirects at this exact path
// Must be served over HTTPS (no ATS exceptions)
{
    "applinks": {
        "details": [{
            "appID": "TEAMID.com.yourcompany.yourapp",
            "paths": ["/path/*", "/other-path"]
        }]
    }
}
```

### Common failures
- AASA file behind authentication (CDN, firewall)
- AASA file returns 301 redirect instead of direct response
- Wrong `appID` format (must include Team ID prefix)
- AASA file not at `.well-known/apple-app-site-association` exact path
- Server doesn't return `Content-Type: application/json`

### Verification

```bash
# Check if Apple can reach your AASA file
curl -v "https://app-site-association.cdn-apple.com/a/v1/yourdomain.com"

# Or use Apple's diagnostic tool:
# https://search.developer.apple.com/appsearch-validation-tool/
```

---

## Widget, Live Activity, and App Clip Rules (§2.5.16)

### Widgets

> *"Widgets, extensions, and notifications should be related to the content and functionality of your app."*

Rejection triggers:
- Widget that displays advertising
- Widget unrelated to host app's core function
- Widget that deep-links to purchase flows
- Widget that duplicates another widget without differentiation

### Live Activities

- Must represent real-time data (sports scores, delivery tracking, timers)
- Cannot be used for persistent advertising
- Must end when the activity is complete (don't leave stale Live Activities)
- Dynamic Island content must be concise and glanceable

### App Clips

- Cannot contain advertising (§2.5.16(a))
- Must include all features in the main app binary
- Size limits enforced strictly (10MB–100MB depending on iOS version)
- Must provide genuine value in the clip experience
- Cannot require account creation for basic App Clip functionality

---

## Accessibility as a Rejection Vector

While Apple doesn't have an explicit "accessibility" guideline that causes rejection, the **Human Interface Guidelines** strongly recommend accessibility, and several guidelines indirectly enforce it:

- §4.2 Minimum Functionality: An app with inaccessible UI may be considered "not app-like"
- §2.1 App Completeness: VoiceOver causing crashes = completeness failure
- §5.1.1 Privacy: Users who deny permissions must still be able to use the app

### Best practices that reduce rejection risk

```swift
// Ensure all interactive elements have accessibility labels
Button(action: {}) {
    Image(systemName: "plus")
}
.accessibilityLabel("Add new item")

// Ensure Dynamic Type support (reduces §4.2 risk on iPad)
Text("Welcome")
    .font(.title)  // System fonts automatically support Dynamic Type

// Support VoiceOver navigation order
VStack {
    headerView
    contentView
    footerView
}
.accessibilityElement(children: .contain)
```

---

## Commission Rate Changes (2026)

### China (effective March 15, 2026)
- Standard IAP commission: **25%** (down from 30%)
- Small Business Program rate: **15%** (was 15%, unchanged)
- Auto-renewal rate after year 1: **20%** (down from 25%)

### Global rates (unchanged)
- Standard: 30% (27% EU with Core Technology Fee alternative)
- Small Business Program (< $1M revenue): 15%
- Auto-renewal after year 1: 15%

### EU Digital Markets Act (DMA) alternatives
- Core Technology Fee: €0.50 per first annual install over 1M (in lieu of reduced commission)
- Alternative payment processing: -3% commission reduction
- Alternative distribution via notarization: different fee structure

---

## Legal Precedent: Apple's Right to Remove (March 2026)

**Source:** https://www.macrumors.com/2026/03/18/apple-wins-decisive-victory-in-musi-app-store-removal-lawsuit/

A federal judge ruled that Apple's developer agreement gives it the right to **remove any app from the App Store at any time, "with or without cause."**

Implications for developers:
- Appealing a removal is a business request, not a legal right
- The developer agreement supersedes the published guidelines
- Apple can remove apps even if they comply with all published guidelines
- This reinforces the importance of maintaining a positive relationship with App Review

---

## Overlooked Permission: Local Network Access

iOS 14+ requires user permission for local network discovery (Bonjour, mDNS, multicast).

### Rejection pattern
- App requests local network without a clear purpose string
- Reviewer denies the prompt and app becomes non-functional

### Info.plist requirement

```xml
<key>NSLocalNetworkUsageDescription</key>
<string>Discover and connect to smart home devices on your Wi-Fi network.</string>

<!-- Also declare Bonjour services if applicable -->
<key>NSBonjourServices</key>
<array>
    <string>_http._tcp.</string>
    <string>_myservice._tcp.</string>
</array>
```

---

## Notification Permission Timing

A frequently overlooked rejection trigger: requesting notification permission at app launch.

### Rejected pattern
```swift
// ❌ Requesting at launch — before user sees any value
func application(_ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .badge, .sound]) { _, _ in }
    return true
}
```

### Correct pattern
```swift
// ✅ Request in context, after user has seen value
struct OnboardingCompleteView: View {
    var body: some View {
        VStack {
            Text("Stay updated on your orders")
            Text("Get notified when your items ship or are delivered.")
            
            Button("Enable Notifications") {
                UNUserNotificationCenter.current().requestAuthorization(
                    options: [.alert, .badge, .sound]
                ) { granted, _ in
                    // Continue regardless of user choice
                }
            }
            
            Button("Not Now") {
                // Continue without notifications — app must work normally
            }
        }
    }
}
```

### Apple's review behavior
Reviewers test what happens when they **deny** every permission prompt. If the app:
- Crashes → §2.1 rejection
- Shows a blocking error screen → §5.1.2 rejection (coercion)
- Shows empty state with "enable notifications to continue" → rejection
- Continues with reduced functionality → approved

---

## Guideline 5.6.4: App Quality (Post-Approval Risk)

A frequently overlooked post-approval guideline:

> *"Customers expect the highest quality from the App Store... Indications that this expectation is not being met include excessive customer reports about concerns with your app, such as negative customer reviews, and excessive refund requests."*

This means:
- Even approved apps can be removed for low quality
- High refund rates trigger review
- Many 1-star reviews with "scam" or "doesn't work" flag the app
- Apple monitors post-approval quality metrics

### Proactive defense
- Monitor crash rates via App Store Connect → Analytics → Crashes
- Respond to negative reviews via ASC (shows Apple you're engaged)
- Fix reported issues promptly in updates
- Don't incentivize positive reviews (§3.2.2(x) violation)
