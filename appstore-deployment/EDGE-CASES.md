# App Store Edge Cases, AI Policy, Permissions, and Code Execution

Covers scenarios where policies are ambiguous, frequently misunderstood, or where community success patterns have emerged despite seeming contradictions with guidelines.

---

## AI Features In Apps — Complete Policy Map

### Apple's explicit AI-related rules (§5.1.2(i), §1.2, §4.7)

Apple does not ban AI features. The specific rules that apply:

**Data disclosure for AI (§5.1.2(i)):**  
> *"You must clearly disclose where personal data will be shared with third parties, **including with third-party AI**, and obtain explicit permission before doing so."*

This means: if your app sends user data to OpenAI, Anthropic, Google Gemini, Mistral, or any LLM provider, you must:
1. Disclose this in your privacy policy (naming the provider)
2. Obtain explicit user consent before transmitting personal data
3. Declare this in the App Privacy nutrition label as "Data Shared with Third Parties"

**On-device vs. server-side AI:**
- On-device inference (Core ML, Apple Intelligence, local LLM): data doesn't leave the device → lower privacy burden
- Server-side inference: treated as data transmitted off-device → full privacy disclosure required

### AI-generated content and UGC rules (§1.2)

If AI generates content that users can see (chatbot responses, generated images, AI-written posts):
- Treat the AI output as UGC for moderation purposes
- Must have: report mechanism, filter mechanism, block mechanism, contact info
- AI chatbots that can discuss inappropriate topics must include content filtering
- Age-gating required if AI can produce mature content

**Rejection pattern:** App with AI chatbot rejected because reviewer prompted it to produce harmful content and there was no filtering or reporting mechanism.

### On-device AI and Apple Intelligence (iOS 18+)

- Apple Intelligence APIs: no explicit consent needed beyond standard permission (data stays on device)
- Writing tools, image generation via Apple Intelligence: treated as on-device processing
- Custom on-device Core ML models: still requires declaring what user data the model processes

### Code generation apps and §2.5.2

**Gray zone that Apple addresses via §2.5.2 exception:**

> *Allowed (educational exception)*: App that lets users write, view, and edit code. The code must be viewable/editable by the user and not used to change the app's own behavior.

- ✅ Coding tutorial app where users write and run sandboxed code → OK
- ✅ Swift Playgrounds-style sandbox → OK  
- ✅ AI coding assistant generating code the user sees and manually applies → OK
- ❌ App that generates code and auto-applies it to change the app's behavior → §2.5.2 violation
- ❌ AI agent that modifies the running app's logic at runtime → rejected

**BREAKING — March 2026 enforcement:**  
Apple removed "Anything" (a $100M-valued vibe coding app) from the App Store on March 26, 2026, and blocked updates to Replit and Vibecode under §2.5.2. Apple stated there are "no specific rules against vibe coding" but apps must not download, install, or execute code that introduces or changes features. Even previewing generated code in a web browser (instead of native execution) was rejected when Anything attempted to comply. This is active, escalating enforcement — see [ADDENDUM.md](ADDENDUM.md) for full details.

Apple has been increasingly scrutinous about "AI agent" apps that claim to perform autonomous device actions. If your app uses any framework that executes code on behalf of the user to control device behavior (not just generate text), expect additional scrutiny.

### AI content generation in Kids Category

- Absolutely zero third-party AI providers that retain data
- On-device AI may be acceptable if no personal data involved
- Content filtering must be extremely robust — Apple holds children's apps to much higher standard

### AI + age-rating update edge case (2026)

Apple's 2026 age-rating update introduces a practical requirement for creator-style AI apps: if users can generate or encounter content that can exceed the app's age rating, the app needs more than just an App Store metadata answer. Apple now expects creator apps to:
- let users identify content that exceeds the app's age rating, and
- restrict underage access using verified or declared age.

This creates a new risk for AI chat, image, avatar, and roleplay apps that previously relied only on "17+" store metadata.

---

## App Store vs Notarization Scope (Frequently Overlooked)

Apple now documents multiple distribution channels (App Store, EU/Japan alternative marketplaces, web distribution for eligible regions).  
For this skill, treat **App Store Review Guidelines as authoritative for App Store submissions** and do not assume notarization checks are identical.

**Practical rule:**
- If the team ships to App Store + alternative channels, run two compliance profiles:
  1) App Store profile (full guideline checks),
  2) Notarization/alternative distribution profile (channel-specific requirements).

This avoids false positives/false negatives when teams copy one checklist across all channels.

---

## Permissions: Complete Edge Case Map

### The "coercion" rule (§5.1.2(i))

> *"Your app may not require users to enable system functionalities (e.g. push notifications, location services, tracking) in order to access functionality, content, use the app, or receive monetary or other compensation."*

**Common violations:**
- "Enable location to continue using the app" as a hard gate (even if location makes the feature better)
- "Allow notifications to access premium content" as a condition of subscription
- Offering gift cards or discounts in exchange for enabling ATT tracking

**Correct pattern:**
```swift
// ❌ Wrong — blocking UI that gates access on permission grant
if locationAuthorizationStatus == .notDetermined {
    showLocationRequiredScreen() // blocks app usage
    return
}

// ✅ Correct — ask, then degrade gracefully if denied
func requestLocationIfNeeded() {
    locationManager.requestWhenInUseAuthorization()
    // Continue app flow regardless of user choice
}

// When permission is denied: show reduced functionality, not a wall
func handleLocationDenied() {
    showNearbyFeatureUnavailableState() // feature-level degradation, not app-level block
}
```

### The "pre-permission screen" pattern (best practice, not required)

Many apps show a custom screen before the system permission dialog to explain context. Apple allows this as long as it's truthful and not deceptive.

```swift
struct LocationPermissionExplainerView: View {
    let onContinue: () -> Void
    let onSkip: () -> Void
    
    var body: some View {
        VStack(spacing: 24) {
            Image(systemName: "map.circle.fill")
                .font(.system(size: 60))
            
            Text("Find places near you")
                .font(.title2.bold())
            
            Text("Location is used only while you're browsing the map. We never track you in the background or share your location with third parties.")
                .multilineTextAlignment(.center)
                .foregroundStyle(.secondary)
            
            Button("Enable Location", action: onContinue)
                .buttonStyle(.borderedProminent)
            
            Button("Not Now", action: onSkip)  // Must always allow skipping
                .foregroundStyle(.secondary)
        }
    }
}
```

### Sensitive permissions that trigger extra scrutiny

| Permission | Apple's scrutiny level | Common rejection pattern |
|------------|----------------------|-------------------------|
| Location (always) | Very high | Background location without clear necessity |
| Camera | High | Purpose string doesn't explain specific use |
| Microphone | High | App records without user knowing |
| Contacts | High | Building contact database for developer |
| Health (HealthKit) | Extreme | Not directly relevant to app's stated purpose |
| HomeKit | Extreme | Not directly relevant to smart home function |
| Face ID/biometrics | High | Used for purposes beyond authentication |
| Background modes | High | Claiming modes not actually used |

### Background mode misuse

Requesting background modes your app doesn't actually need is a rejection trigger.

```xml
<!-- Info.plist — only include modes you actually use -->
<key>UIBackgroundModes</key>
<array>
    <!-- Only include what applies: -->
    <!-- audio -->
    <!-- fetch -->
    <!-- location -->
    <!-- remote-notification -->
    <!-- processing -->
    <!-- voip -->
</array>
```

Requesting `location` background mode when app only needs it while in foreground → rejection with message about unnecessary background usage.

---

## Scam / Dark Pattern Enforcement

### Apple now treats trust failures as first-class enforcement risk

The skill previously covered subscription clarity, but Apple goes further under the Developer Code of Conduct:
- apps must not prey on users,
- trick them into unwanted purchases,
- force unnecessary data sharing,
- raise prices in a tricky manner,
- or charge for features/content not delivered.

This is broader than classic guideline-matching. Even if a flow appears technically compliant, it can still be rejected or removed if it looks scammy.

### Patterns that are especially dangerous

- Free-trial screens where the billed amount is visually de-emphasized
- Fake countdown timers or false scarcity
- Hiding subscription management / cancel paths
- Nudging users into ATT consent with rewards or access unlocks
- Promising impossible AI/device capabilities such as malware scanning, diagnostics, or \"autonomous device control\"
- Selling access to OS primitives like push notifications or camera usage

### Safe rule

Whenever a flow involves billing, consent, or user data:
1. Put the real consequence first
2. Make the billed amount or data-sharing consequence explicit
3. Keep opt-out / restore / manage paths close at hand
4. Avoid copy that creates urgency unless the deadline is real and externally enforced

---

## EU Digital Services Act (trader) — operational edge case

This is **App Store Connect compliance**, not a paragraph in the binary review guidelines, but it still **stops releases**:

- Apple must collect and verify **trader contact data** for EU distribution (address, phone, email on the product page).
- **Non-trader / hobbyist** apps may declare that status; consumers are then informed that certain EU consumer-contract rights may not apply.
- **Per-app** overrides exist — one account can be a trader for a commercial app and non-trader for a side project.
- Documentation uploads (business address proof, etc.) can take time; complete **before** a hard launch date.

Primary: [Manage EU DSA trader requirements](https://developer.apple.com/help/app-store-connect/manage-compliance-information/manage-european-union-digital-services-act-trader-requirements/).

---

## Alternative distribution & Notarization (EU / Japan)

If the same codebase ships **outside** the classic App Store (alternative marketplaces, Web Distribution, Japan marketplace rules), Apple applies **Notarization** and a **highlighted subset** of the App Review Guidelines. Malware scanning and safety rules still apply. Policy can **diverge** from full App Store review (e.g. different entitlement programs). Always cross-check the guidelines page with **"Highlight Notarization Review Guidelines Only"** and [DMA / EU support](https://developer.apple.com/support/dma-and-apps-in-the-eu/).

---

## Code Download & Hot Updates — Complete Policy Map

### The §2.5.2 spectrum

```
ALLOWED ←————————————————————————————→ REJECTED
  |                                          |
Content   Config    Server-    Educational  Feature    Hot
updates   updates   driven UI  code sandbox changes    patches
(text,    (JSON     (layout    (user        (new       (new JS
images)   flags)    varies)    sees code)   screens)   bundle)
```

### Allowed server-driven patterns

```swift
// ✅ Config-driven UI (layout is fixed, data varies)
struct RemoteConfigManager {
    struct AppConfig: Decodable {
        var accentColor: String
        var homeTabOrder: [String]
        var featureFlags: [String: Bool]  // only shows/hides existing features
        var promotionalBannerText: String?
    }
    
    func fetchConfig() async throws -> AppConfig {
        // Fetch from server — this is allowed
        // The returned config changes presentation, not code
    }
}

// ✅ Content-driven updates (articles, products, images) — always OK
// ✅ Feature flag gating (show/hide built-in views) — OK if not hiding during review
// ✅ A/B testing via remote config of content/copy — OK
// ✅ Paywall remote config (changing prices, copy) — OK if no new code executed
```

### Rejected hot-update patterns

```javascript
// ❌ React Native CodePush downloading new JavaScript bundle
import CodePush from 'react-native-code-push';

CodePush.sync({
    updateDialog: true,
    installMode: CodePush.InstallMode.IMMEDIATE  // ← This is what Apple rejects
});

// ❌ Downloading and eval()-ing remote JavaScript
fetch('https://myserver.com/logic.js')
    .then(r => r.text())
    .then(code => eval(code));  // ← Absolute rejection

// ❌ Dynamic framework loading from remote URL (iOS blocks this anyway via security)
```

### Safe React Native / Expo configuration

```json
// Expo app.json — disable OTA updates that change functionality
{
  "expo": {
    "updates": {
      "enabled": false
    },
    "runtimeVersion": {
      "policy": "nativeVersion"
    }
  }
}
```

### §4.7 Mini-apps / Chatbots / Plug-ins — Allowed but with full compliance

§4.7 explicitly allows:
- HTML5/JavaScript mini apps and mini games
- Streaming games
- Chatbots
- Plug-ins
- Retro game emulators (can download ROMs user owns)

**BUT they must comply with:**
- Full §5.1 privacy rules (including for the mini-app's own data collection)
- §1.2 UGC moderation requirements (if mini-app has user content)
- §3.1 payment rules (digital goods in mini-apps need IAP)
- Cannot use mini-apps to bypass App Review (feature added post-review via mini-app = rejection)

---

## External Payments — Storefront Nuances (Most Misunderstood Area)

### The contradiction developers encounter

"I heard external payment links are allowed now" → Get rejected anyway.

**Why:** The US storefront exception is real but limited. Most developers distribute globally.

### Current rules by region (Feb 2026)

**United States:**
- External links/CTAs: ✅ No entitlement needed (US storefront only)
- BUT: Apple still shows an information sheet to users before they leave the app
- Must not engage in "misleading marketing practices, scams, or fraud"
- If user leaves to external purchase and returns, app must handle entitlements via server check

**European Union (iOS/iPadOS):**
- StoreKit External Purchase Link Entitlement required
- Limited to specific storefronts per entitlement agreement
- Must use Apple's disclosure sheet before external navigation

**Netherlands (dating apps):**
- Separate entitlement program specific to dating category
- External payment links allowed for dating apps only
- Reference: https://developer.apple.com/support/storekit-external-entitlement/

**South Korea:**
- StoreKit External Purchase Entitlement for South Korea
- Requires separate binary distributed only in South Korea App Store
- Must use required StoreKit APIs

**Everywhere else:**
- External purchase links/CTAs: ❌ Not allowed
- "Reader app" link entitlement: ✅ Available for qualifying reader apps (website link for account management only, not purchase)

### Reader App entitlement — what qualifies

A "reader app" provides access to previously purchased content:
- Streaming music services
- Streaming video services
- Digital book/magazine readers
- Podcast apps
- News/article apps

Does NOT qualify:
- Apps where users make purchases of new digital content inside the app
- Social media apps
- Games

### Free companion app pattern (safe harbor for B2B SaaS)

This is the most reliable approach for SaaS apps avoiding IAP conflict:

```
✅ Structure that works:
- App is free to download
- All features accessible after login (no in-app paywall UI)
- Subscription managed entirely on your website
- App description clearly states "Subscription required; manage at yoursite.com"
- No button, link, or text in the app directing users to purchase

❌ Rejected patterns even in companion apps:
- Any button labeled "Subscribe", "Upgrade", "Get Premium" that opens external URL
- Any price or plan information shown in-app that directs to external checkout
- "View our plans" link that opens pricing page
```

### Underused but valid payment-side tools Apple encourages

- `showManageSubscriptions(in:)` for letting users manage/cancel in-app through Apple's UI
- Billing Grace Period to reduce involuntary churn without access interruptions
- App Store Server Notifications + App Store Server API to keep access state accurate during billing retry and grace periods

If your app fails to restore access correctly after billing recovery or win-back redemption, reviewers can interpret that as poor subscription implementation even when the paywall itself is compliant.

---

## Content Policy Edge Cases

### Explicit content (NSFW) — the website-toggle exception

Apple's guidelines (§1.2) contain an explicit allowance:

> *"If your app includes user-generated content from a web-based service, it may display incidental mature 'NSFW' content, provided that the content is hidden by default and only displayed when the user turns it on **via your website**."*

This is specifically "via your website" — NOT via an in-app setting. The on/off control must be on your web platform.

**Pattern that passes:**
1. NSFW content hidden by default in app
2. User visits website → account settings → enables "Mature Content"
3. App fetches user preference from server → unlocks NSFW content display
4. The toggle control never appears inside the iOS app

### Gambling gray zone

**What looks like gambling but may pass:**
- "Skill-based" competitions with prizes (varies by jurisdiction; get legal review)
- Prediction markets (heavy scrutiny; needs legal compliance)
- Fantasy sports (varies by jurisdiction)

**What definitely doesn't pass:**
- Real-money wagering without proper licensing and geo-restriction
- Simulated casino games that offer real prizes
- Binary options

### Political and satirical content (§1.1.1)

> *"Professional political satirists and humorists are generally exempt"* from the objectionable content rule.

Apps that use satire as a shield for defamatory content will still be rejected. The exception is for genuine satire/commentary, not for targeting specific individuals.

### Medical claims (§1.4.1)

**Always rejected:**
- "Diagnoses [condition] using your iPhone camera"
- "Measures blood pressure with no hardware"
- "AI detects cancer from photos"
- Any claim that requires FDA/CE clearance if you don't have it

**Passes with proper disclosure:**
- "This app is not a medical device and is for informational purposes only"
- "Consult your healthcare provider before making medical decisions"
- Features that track/log health data without making diagnostic claims

---

## Regulated App Categories

### Financial apps (§3.2.1(viii))

- Trading, investing, money management apps: must be submitted by the financial institution
- Must have necessary licensing in all locations distributed
- Personal loan apps: must disclose APR, payment date; cannot exceed 36% APR; cannot require repayment in 60 days or less

### VPN apps (§5.4)

- Organization enrollment required (no individual accounts)
- Must use NEVPNManager API
- Cannot sell/use/disclose user data to third parties
- If distributed in regions requiring VPN license: provide license in App Review Notes

### Cryptocurrency apps (§3.1.5)

- Wallet: organization only
- Mining: off-device only (no on-device mining)
- Exchange: license required by jurisdiction
- ICO/futures: established financial institution only

---

## Sign In With Apple — Edge Cases

### When it is NOT required despite third-party login

These are real exceptions, not loopholes:

1. App uses only your own proprietary login (no OAuth to social platforms)
2. App is an education/enterprise app requiring institutional accounts (Google Workspace for school, Okta for enterprise)
3. App is a client for a specific third-party service where users must sign in to that service directly (Gmail client, Slack client)
4. Government/citizen ID system
5. Alternative marketplace apps

### When reviewers get it wrong (appeal-worthy)

Enterprise apps using Okta/SAML have been rejected with Sign in with Apple requirement. This is a reviewer error — the enterprise exception applies. Include in Notes for Review:
```
Note: This app is an enterprise application distributed to employees of [Company]. 
Users authenticate via [Company]'s enterprise identity provider (Okta/SAML). 
This qualifies as an "education or enterprise app that requires the user to sign 
in with an existing enterprise account" per §4.8 exception criteria.
```

---

## Metadata Traps

### Screenshots showing non-current UI

Apple compares screenshots to the actual app during review. If redesigned UI was shipped but screenshots still show the old design → 2.3 violation.

**Automation recommendation:**
```bash
# Fastlane snapshot to auto-generate screenshots before each submission
fastlane snapshot
```

### "What's New" generic descriptions rejected

**Rejected:** "Bug fixes and performance improvements"  
**Rejected:** "Updated app for better experience"  
**Required:** "Fixed crash when opening Settings on iOS 17.2; improved load time on profile page by 40%; added support for Dynamic Island on iPhone 15 Pro"

Apple guidelines state: *"All new features, functionality, and product changes must be described with specificity in the Notes for Review section... generic descriptions will be rejected."*

### Keywords and title stuffing

Rejected patterns:
- Competitor app names in keywords
- "Free", "Best", "#1" in app name or subtitle
- Excessive keyword repetition
- Category names as keywords (App Store already categorizes your app)

### Age rating metadata is now operational, not just descriptive

Because Apple changed the age-rating system in 2026, stale App Store Connect answers can now interrupt submissions. For apps with UGC or AI generation, re-check age-rating responses when:
- adding chat or image generation,
- allowing mature/user-uploaded content,
- adding social sharing,
- or changing moderation posture.

---

## 4.2 Minimum Functionality Appeal Tactics (When Rejection Is Borderline)

Field evidence (StackOverflow/dev.to) shows some 4.2 rejections resolve after a clear, feature-by-feature reviewer response.

**What tends to work:**
1. Enumerate each native workflow in bullet form (not marketing language).
2. Provide exact in-app navigation path for every claimed feature.
3. Attach short video walking all tabs/screens.
4. Ask a precise clarification question: \"Which specific flow is considered insufficiently app-like?\"

**What tends to fail:**
- Generic appeals (\"our app is useful\"),  
- Arguing popularity/download counts without feature evidence,  
- Re-submitting same binary with no reviewer guidance updates.

**Use-case note:** This is for borderline interpretation disputes. If the app is truly a thin wrapper, add native value and resubmit instead of escalating.

---

## Resolution Center Packet Design (Operational Edge)

When reviewers report issues you cannot reproduce:

**Packet contents:**
- Issue statement mapped to exact guideline clause
- Repro path with numbered steps
- Temporary diagnostics summary (non-PII)
- Screenshot/video attachment
- Binary or metadata change summary

**Why:** review teams can continue conversation with attachments until you resubmit; strong packets reduce repeat cycles.

---

## How Contradictions Get Resolved

When community advice conflicts with Apple's guidelines, apply this resolution order:

1. **Most recent Apple documentation** → Authoritative
2. **Apple WWDC session transcripts** → Authoritative (note session year)
3. **Apple Developer Forum official responses** → Near-authoritative
4. **Apple support pages** → Authoritative
5. **Community consensus from last 12 months** → Hypothesis to test
6. **Older blog posts / guides** → Treat as potentially outdated

**Common contradictions:**
- "External payment links are now allowed" → Partially true; only US storefront, and only if meeting current rules
- "You don't need privacy policy if you don't collect data" → False; always required
- "Subscription apps don't need IAP if B2B" → False unless meeting specific exemption criteria
- "Sign in with Apple isn't required if I use Firebase" → Depends on whether Firebase auth uses third-party social login as primary; if it does, SiwA required

---

## Appeal Strategy

### When to appeal vs. fix and resubmit

| Scenario | Recommendation |
|----------|---------------|
| Reviewer made factual error about your app's behavior | Appeal with evidence (screenshots, video) |
| Rejection based on guideline that has an exception you qualify for | Appeal citing exact exception text |
| Policy is clear and you violated it | Fix and resubmit; don't waste appeal |
| Vague rejection message that doesn't match any guideline | Communicate via Resolution Center first |
| Repeated rejection for same issue | Escalate to App Review Board |

### Appeal message template

```
Re: [App Name] Rejection — Guideline [X.Y]

We respectfully disagree with this rejection for the following reasons:

1. [Guideline X.Y] states: "[exact quote from guideline]"
   Our app [specific behavior]: [description]
   This complies because: [specific reasoning]

2. [If there's an exception that applies]:
   Guideline [X.Y](z) provides an exception for [category].
   Our app qualifies because: [evidence]

We have attached [screenshots/video] showing [specific feature/behavior] 
at [exact navigation path] to document compliance.

We welcome any clarification from App Review.
```

### What appeals cannot fix

- Clear violations of content policies (pornography, illegal content)
- Safety issues (malware, data theft)
- Repeated manipulation of the review process
- Developer Code of Conduct violations

### Process levers developers often overlook

- You can attach screenshots, PDFs, demo videos, and authorizations directly to App Review replies until you resubmit
- Metadata-only rejections can usually be fixed without a new binary
- In multi-item submissions, you can remove the rejected item and continue with the rest
- Apple offers App Review appointments for guidance before or during difficult review cycles
- For already-live apps, Apple may allow a bug-fix update through while non-legal/non-safety issues are deferred to the next submission

---

## Pre-Submission Automated Checks

### Xcode build phase script for common issues

```bash
#!/bin/bash
# Run as a pre-archive build phase

echo "=== App Store Pre-Submission Check ==="

# Check for hardcoded IPv4 addresses
if grep -r "[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}" --include="*.swift" .; then
    echo "⚠️  WARNING: Possible hardcoded IPv4 addresses found. IPv6-only review networks may fail."
fi

# Check for UIWebView (deprecated, rejected)
if grep -r "UIWebView" --include="*.swift" .; then
    echo "❌  ERROR: UIWebView detected. Replace with WKWebView before submission."
    exit 1
fi

# Check for placeholder text
if grep -ri "lorem ipsum\|placeholder\|TODO: remove\|FIXME: remove" --include="*.swift" .; then
    echo "⚠️  WARNING: Possible placeholder content in code."
fi

# Check for prefs:root private URL scheme
if grep -r "prefs:root" --include="*.swift" .; then
    echo "❌  ERROR: Private prefs:root URL scheme detected. Use UIApplicationOpenSettingsURLString."
    exit 1
fi

echo "=== Check complete ==="
```

### Privacy manifest validation

```bash
# Generate privacy report in Xcode
# Product → Archive → then in Organizer: Validate App → Privacy Report

# Or via command line (requires Xcode 15+):
xcodebuild archive \
    -scheme YourScheme \
    -archivePath /tmp/yourapp.xcarchive \
    -destination 'generic/platform=iOS'

# Then validate:
xcrun altool --validate-app \
    -f /tmp/yourapp.xcarchive \
    --type ios \
    --apiKey YOUR_KEY \
    --apiIssuer YOUR_ISSUER
```

---

## The "Gatekeeper Game" — How Apple's System Actually Works

From Apple's own App Review team talks and community evidence:

**What reviewers actually do:**
1. Test on physical device (not Simulator)
2. Follow instructions in App Review Notes
3. Test like a user, not a developer
4. Check the paywall purchase flow
5. Check privacy policy link works
6. Check account deletion path
7. Run automated binary analysis (private APIs, privacy manifests)
8. Evaluate whether pricing, consent, and account flows feel trustworthy and understandable

**What triggers extra scrutiny:**
- First submission of a new category/concept
- Significant new features in an update
- Subscription model being introduced
- App with UGC or social networking
- Health/medical claims
- Kids Category apps
- Financial apps
- Apps with aggressive subscription paywalls or fast-moving pricing experiments
- Creator / AI apps with open-ended generation
- Apps submitted by an entity that doesn't clearly match the regulated service provider

**What reduces scrutiny:**
- Long history of clean submissions from the same developer account
- Detailed, accurate App Review Notes
- Quick responses to reviewer questions via Resolution Center
- No history of manipulation or violations

**The "30% rule":**  
Apple's App Review team has stated that tips to avoid common issues affect "30 percent of rejected submissions" — meaning detailed notes and proper demo access prevents roughly 1-in-3 rejections.
