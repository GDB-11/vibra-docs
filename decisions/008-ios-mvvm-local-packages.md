# 008 — iOS: MVVM + repositories, local Swift packages

**Status:** Decided, including minimum iOS version (**latest iOS**, 2026-11-23). Run in the Xcode Simulator only.

## Context
SwiftUI app with playback, store, widget and offline features that must stay decoupled.

## Choice
SwiftUI, Swift 6 strict concurrency, async/await, `@Observable`. Layers: `View → ViewModel (@Observable, @MainActor) → Repository (protocol in Domain) → Networking + Persistence`. Features depend on `Domain` and `DesignSystem`, never on each other. Navigation through a coordinator/router in the app target.

```
Vibra/                       ← thin target: composition root, DI, navigation
Packages/
  Domain/  Networking/  Persistence/ (SwiftData or GRDB)  Playback/  Store/  DesignSystem/
  Features/  Auth Home Search Library Player Store Subscription Settings
```

## Technical points
- **Playback:** global `PlaybackService` (wraps `AVPlayer`/`AVQueuePlayer`), independent of views. `AVAudioSession` `.playback` + audio background mode; `MPNowPlayingInfoCenter` / `MPRemoteCommandCenter`; handle interruptions and route changes.
- **Offline:** `URLSession` background configuration (`AVAssetDownloadURLSession` if HLS). Store in SwiftData the entitlement each download came from (subscription downloads lock on expiry; purchased ones are kept).
- **Testing:** Swift Testing for ViewModels/repositories; a few XCUITests (login → play).
- **Localization:** strings via `String(localized:)`; UI in Spanish.

## Open items
- **To verify:** the exact version number of "latest iOS" against the installed Xcode when the project is created. This replaces the earlier "one below current" suggestion.
