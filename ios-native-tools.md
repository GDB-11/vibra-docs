# The 3 native iOS tools (course requirement)

| # | Tool | What to implement | What to research and document |
|---|---|---|---|
| 1 | **StoreKit 2** | Monthly/annual subscription, album and track purchase, paywall, `Transaction.updates`, restore purchases, renewal/expiration states | Differences from StoreKit 1, product types, transaction verification, testing with `StoreKitTest`, guideline 3.1.1 |
| 2 | **AVFoundation + MediaPlayer** | Background `AVPlayer`, `AVAudioSession`, lock screen and Control Center with `MPNowPlayingInfoCenter`/`MPRemoteCommandCenter`, interruptions | Audio session lifecycle, HLS vs progressive download, audio routes (AirPods/Bluetooth), why the player lives outside the views |
| 3 | **WidgetKit + ActivityKit** | "Now playing" widget and Live Activity (lock screen and Dynamic Island) driven from the app | Widget refresh budget, app–extension data sharing, Live Activity limits, local vs push-updated |

**Reserve:** App Intents (Siri, Shortcuts, Spotlight) if time runs short or if the course counts WidgetKit and ActivityKit as one. *Idea, not a decision.*

**Dropped to avoid paid-account or special-permission dependencies:** [APNs](decisions/r03-apns-push.md), [CarPlay](decisions/r05-carplay.md), [Sign in with Apple](decisions/r04-sign-in-with-apple.md).

**Confirmed (2026-11-23):** these 3 tools are accepted for the course. The app runs only in the Xcode Simulator, so free-account limits for App Groups and Background Modes no longer apply.
