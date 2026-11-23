# Name and descriptions

**Decision 015 — Name:** Vibra. **Status:** Decided.
**Tagline:** *Todo el catálogo, o solo lo que amas.*

**Short description (user-facing, shown in Spanish in the app):**
> **Vibra** is a music streaming app for iOS with two ways to enjoy music: subscribe (monthly or annual) to access the whole catalog, or buy songs and albums to keep forever. It plays in the background, can be controlled from the lock screen, and shows what is playing in a widget and the Dynamic Island.

**Technical description (README):**
> **Vibra** is a demo music streaming project made of a native SwiftUI iOS app and an ASP.NET Core 11 backend built as a modular monolith.
>
> **What it demonstrates**
> - **Streaming:** audio served from object storage through signed URLs, with background playback.
> - **Simulated monetization:** auto-renewable subscriptions and perpetual purchases with StoreKit 2, and a backend entitlements module that decides what each user can listen to, with 30 s previews for locked content.
> - **Native iOS tools:** StoreKit 2, AVFoundation + MediaPlayer, and WidgetKit + ActivityKit.
>
> **Stack:** SwiftUI · Swift 6 · StoreKit 2 · AVFoundation · WidgetKit · ASP.NET Core 11 · Dapper · PostgreSQL · FFmpeg · OpenAPI.
>
> *Educational project. Payments are simulated and the music is freely licensed.*

Before publishing anything under the name "Vibra", check availability on the App Store and GitHub (short names are often taken). Not an issue for a course demo.
