---
layout: devlog
title: "Trust Insights: Detecting Coerced Actions in iOS 27"
category: devlog
source: https://developer.apple.com/videos/play/wwdc2026/379/
tags: [ios, security, swift, fraud]
---
Summary of WWDC2026 session about:
Trust Insights (iOS 27) — behavioral coercion detection framework.

**The problem it solves:** Social engineering attacks where the user performs the action themselves (authenticated, legitimate), so MFA/biometrics are useless. Need a signal for coerced vs. genuine intent.

**Integration flow:**
- Requires an entitlement/capability in Xcode
- Client-side Swift API only (no server integration needed)
- Create `InsightContext` with `operationCategory` (payment, account, resourceUse, communication, other) + `InsightEvaluation`
- Call `requestEvaluation()` async — takes a few seconds, needs internet
- Check authorization status first (user can disable it in Settings)

```swift
guard await InsightContext.authorizationStatus == .authorized else { return }

let context = InsightContext(
    operationCategory: .payment,
    evaluation: InsightEvaluation()
)
let insight = try await context.requestEvaluation()

switch insight.isLikelyBeingCoached {
case .high:    showCoercionWarning()
case .medium:  requireAdditionalVerification()
case .unknown: break  // no evidence — not a green light
}

insight.reportConsumption()  // mandatory
```

**Result values for `IsLikelyBeingCoachedInsight`:**
- `unknown` — no evidence, but not to be treated as low risk
- `medium` — introduce friction, additional verification, adjust risk scoring
- `high` — warn the user before proceeding

**Feedback (mandatory):**
- `reportConsumption()` must be called per evaluation or you get rate-limited
- Offline fraud labels submitted via Apple Business Register (server-to-server) — optional but helps the model

**Privacy architecture:**
- Device-sourced signals (interaction patterns, timing, sensors) processed locally, inputs discarded immediately
- Only the single output value leaves the device
- Apple Account signals + velocity checks may be incorporated server-side
- Never touches Photos, Messages, Mail content
- User can disable in Settings (cooldown applies to prevent coaching into disabling it)

**Best practices:**
- Use it at high-value moments: P2P payments, irreversible actions, remote access grants, sensitive data sharing
- Never sole decision factor — integrate into existing risk logic
- Never treat unknown or missing value as safe
- Test with Xcode scheme launch argument overrides (sandbox in dev, production on App Store)

Related: App Attest for verifying requests come from legitimate app instances.
