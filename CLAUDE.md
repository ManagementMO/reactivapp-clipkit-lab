# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ReactivChallengeKit is an iOS SwiftUI App Clip simulator for Hack Canada 2026. It provides a framework for building URL-invoked, ephemeral, single-task experiences that deliver value in under 30 seconds — without deploying to a real App Clip target.

The active submission is **ClipCheck** — a restaurant health inspection scoring clip that shows trust scores, inspection timelines, AI safety recommendations (Gemini), and voice briefings (ElevenLabs TTS).

## Build & Run

Zero external dependencies. No SPM, CocoaPods, or Carthage.

```bash
# Open in Xcode (Cmd+R to build and run on iPhone simulator)
open reactiv_stuff/ReactivChallengeKit/ReactivChallengeKit.xcodeproj

# Command-line build
xcodebuild -project reactiv_stuff/ReactivChallengeKit/ReactivChallengeKit.xcodeproj \
  -scheme ReactivChallengeKit -sdk iphonesimulator \
  -destination 'platform=iOS Simulator,name=iPhone 16' build
```

- **Xcode 26+**, **iOS 16+**, **Swift 5.0**
- No tests or linting configured
- Build script auto-runs `scripts/generate-registry.sh` to regenerate `SubmissionRegistry.swift` and `GeneratedSubmissions.swift`

## Architecture

All source lives under `reactiv_stuff/ReactivChallengeKit/ReactivChallengeKit/`.

### Core Protocol (`Protocol/`)

**`ClipExperience`** — every clip conforms to this. It extends `View`:
```swift
protocol ClipExperience: View {
    static var urlPattern: String { get }
    static var clipName: String { get }
    static var clipDescription: String { get }
    static var teamName: String { get }           // defaults to "Reactiv"
    static var touchpoint: JourneyTouchpoint { get } // defaults to .utility
    static var invocationSource: InvocationSource { get } // defaults to .qrCode
    init(context: ClipContext)
}
```

**`ClipContext`** — passed to clips at invocation, contains `invocationURL`, `pathParameters`, and `queryParameters`.

### Simulator Framework (`Simulator/`)

- **`ClipRouter.swift`** — `@Observable` URL pattern matching and routing. `allExperiences` = built-in + `SubmissionRegistry.all`. Pattern syntax: `:param` (e.g., `example.com/hello/:name`). Host matching is case-insensitive, strips `www.`.
- **`SimulatorShell.swift`** — Root container: landing screen vs active clip, constraint banner, moment timer overlay.
- **`InvocationConsole.swift`** — URL input replacing real QR/NFC triggers.
- **`MomentTimer.swift`** — Elapsed timer: green (<20s), yellow (20-30s), red (>=30s).

### Submission System

```bash
# Create a new submission scaffold
bash scripts/create-submission.sh "Team Name"

# Registry is auto-generated at build time — no manual registration needed
```

Submissions live in `Submissions/<team-slug>/`. Files are compiled via auto-generated `GeneratedSubmissions.swift` (not via Xcode target membership).

### View Hierarchy

```
ReactivChallengeKitApp → SimulatorShell
  ├── LandingView (no active clip)
  │   ├── InvocationConsole
  │   └── InvocationCard per registered clip
  └── ClipHostView (active clip)
      ├── MomentTimer + touchpoint pill + dismiss button
      ├── clip.body (your experience — use ScrollView to avoid overlap)
      └── ConstraintBanner ("Get the full app")
```

### Reusable Components (`Components/`)

`ClipHeader`, `ClipActionButton` (styles: `.primary`, `.secondary`, `.destructive`), `ClipSuccessOverlay`, `ClipBackground`.

## ClipCheck Submission

Files in `Submissions/clipcheck/`:

| File | Purpose |
|------|---------|
| `ClipCheck.swift` | Main experience entry point |
| `RestaurantModels.swift` | Data models |
| `RestaurantDataStore.swift` | Local JSON data loading |
| `GeminiService.swift` | Gemini API for AI safety summaries |
| `ElevenLabsService.swift` | ElevenLabs TTS for voice briefings |
| `MenuAnalysisService.swift` | Menu analysis logic |
| `DietaryProfile.swift` | User dietary preferences |
| `QRScannerView.swift` / `QRGeneratorView.swift` | QR code handling |
| `Secrets.swift` / `Secrets.plist` | API keys |

**URL Pattern:** `example.com/restaurant/:restaurantId/check`
**Test URL:** `example.com/restaurant/baba-chicken-grill/check`

### Demo Flow (Critical Path)

1. Enter URL in InvocationConsole → ClipCheck opens
2. Trust score gauge animates (0-100, color-coded)
3. Inspection timeline with color-coded dots
4. Tap → violation details expand
5. AI Advisor card shows Gemini recommendation
6. Voice button → ElevenLabs reads summary

### Design System

- White/light background, medical/safety aesthetic
- Trust colors: Green `#22C55E` (safe), Amber `#F59E0B` (caution), Red `#EF4444` (danger)
- SF Pro, rounded corners, subtle shadows

### External APIs

- **Gemini:** `POST https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=API_KEY`
- **ElevenLabs:** `POST https://api.elevenlabs.io/v1/text-to-speech/{voice_id}` — returns audio data for AVFoundation

## App Clip Constraints

- URL-invoked only (no app icon launch)
- Ephemeral: no persistent storage, no login, no onboarding
- Single focused task, value in ≤30 seconds
- 15 MB size limit (use SF Symbols and server-loaded images)
- No background processing

## Judging Criteria

| Criteria | Weight |
|----------|--------|
| Novelty of use case | 30% |
| Constraint awareness | 25% |
| Real-world trigger quality | 20% |
| Execution / demo | 15% |
| Scalability of idea | 10% |
