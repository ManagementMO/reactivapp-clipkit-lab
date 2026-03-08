# ClipCheck

**Scan any restaurant. Know before you eat.**

ClipCheck is an App Clip that lets anyone scan a QR code at a restaurant door and instantly see its health inspection score, AI-powered safety analysis, and personalized meal recommendations — no download, no account, no friction.

Built for [Hack Canada 2026](https://hack-canada-2026.devpost.com/) on the [Reactiv ClipKit](https://github.com/reactivapp/reactivapp-clipkit-lab) framework.

---

## Demo

**Test URL:** `example.com/restaurant/baba-chicken-grill/check`

**With personalization:** `example.com/restaurant/baba-chicken-grill/check?allergens=nuts,dairy&diet=vegetarian`

Enter either URL in the InvocationConsole or scan a generated QR code.

---

## What It Does

| Feature | Description |
|---------|-------------|
| **Trust Score** | Animated 0–100 gauge computed from weighted inspection history. Color-coded: green (safe), amber (caution), red (avoid). |
| **Inspection Timeline** | Interactive horizontal timeline of every inspection with pass/conditional/closed status dots. |
| **Violation Details** | Expandable cards for each infraction with severity badges (Minor, Significant, Crucial). |
| **AI Safety Advisor** | Gemini analyzes violation patterns and delivers a plain-English safety assessment with risk level. |
| **Personalized Recommendations** | AI recommends what to order and what to avoid based on violations, weather, time of day, allergens, and dietary preferences. |
| **Voice Briefing** | ElevenLabs reads the full personalized safety summary aloud. One tap, hands-free. |
| **Nearby Alternatives** | If the score is low, surfaces higher-rated restaurants within walking distance. |
| **QR Scanner** | Built-in camera scanner for physical QR codes at restaurant entrances. |

**Value delivered in under 15 seconds. No app download. No account. No login.**

---

## The Personalization Engine

The Reactiv challenge asks: *"How do you personalize meaningfully with no user history?"*

ClipCheck uses **four context signals** — zero stored data:

| Signal | Source | How It's Used |
|--------|--------|---------------|
| **Weather** | [Open-Meteo API](https://open-meteo.com/) (free, no key) | Cold weather favors hot dishes; hot weather raises risk for improperly stored items |
| **Time of Day** | `Date()` — always available | Lunch rush = fresher food; mid-afternoon = may have been sitting longer |
| **Allergens & Diet** | URL query parameters in the QR code | Flags allergen-related violations, filters recommendations, warns about cross-contamination |
| **Violation Patterns** | Inspection history analysis | Identifies systemic issues (e.g., recurring temperature violations) vs. one-off problems |

Two people scanning the same restaurant at different times with different allergens get meaningfully different advice.

---

## Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Framework | SwiftUI on Reactiv ClipKit | App Clip simulator |
| AI Analysis | Gemini API (gemini-3-flash-preview) | Safety assessment + personalized recommendations |
| Voice | ElevenLabs TTS API | Voice briefing with AVSpeechSynthesizer fallback |
| Weather | Open-Meteo API | Current conditions for contextual recommendations |
| QR Scanning | AVFoundation | Camera-based QR code detection |
| QR Generation | CoreImage | Printable demo QR codes |
| Data | Region of Waterloo + Toronto DineSafe | Real public health inspection records |

**Zero external dependencies.** No SPM, no CocoaPods, no Carthage. Everything uses iOS built-in frameworks.

---

## Getting Started

### Prerequisites

- **Xcode 26+**
- **iOS 26+ Simulator** (iPhone 17 Pro recommended)
- macOS with Swift 5.0+

### Run

```bash
# Clone and open
git clone <repo-url>
cd ClipKit
open reactiv_stuff/ReactivChallengeKit/ReactivChallengeKit.xcodeproj
```

Press **Cmd+R** in Xcode to build and run.

### Command-Line Build

```bash
xcodebuild -project reactiv_stuff/ReactivChallengeKit/ReactivChallengeKit.xcodeproj \
  -scheme ReactivChallengeKit -sdk iphonesimulator \
  -destination 'platform=iOS Simulator,name=iPhone 17 Pro' build
```

### API Keys (Optional)

ClipCheck works without API keys — all features have fallbacks. To enable live AI and voice:

1. Copy `Secrets.example.plist` to `Secrets.plist`
2. Add your Gemini and ElevenLabs API keys
3. Rebuild

---

## Project Structure

```
reactiv_stuff/ReactivChallengeKit/ReactivChallengeKit/
├── Submissions/                    # ClipCheck source files (compiled by Xcode)
│   ├── ClipCheck.swift             # Main experience + all UI components
│   ├── RestaurantModels.swift      # Data models (Restaurant, Inspection, Infraction)
│   ├── RestaurantDataStore.swift   # Data loading + trust score computation
│   ├── GeminiService.swift         # Gemini API — AI safety analysis
│   ├── MenuAnalysisService.swift   # Gemini API — personalized menu recommendations
│   ├── WeatherService.swift        # Open-Meteo API — current weather
│   ├── PersonalizationContext.swift # Combines weather + time + dietary signals
│   ├── DietaryProfile.swift        # Allergen & dietary preference models + URL parsing
│   ├── ElevenLabsService.swift     # ElevenLabs TTS + AVSpeechSynthesizer fallback
│   ├── QRScannerView.swift         # AVFoundation camera QR scanner
│   ├── QRGeneratorView.swift       # Printable QR code generator
│   ├── restaurants.json            # Real inspection data (Waterloo + Toronto)
│   ├── Secrets.swift               # API key loader
│   └── Secrets.plist               # API keys (not committed)
├── Protocol/                       # ClipExperience protocol definition
├── Simulator/                      # ClipKit simulator shell
└── Components/                     # Reusable UI components
```

---

## Fallback Architecture

ClipCheck never breaks. Every external dependency has a graceful fallback:

| If this fails... | ClipCheck does this |
|---|---|
| Gemini API unreachable | Generates safety assessment from raw inspection data using a local rules engine |
| ElevenLabs API down | Falls back to iOS AVSpeechSynthesizer |
| Open-Meteo weather fails | Uses hardcoded Waterloo conditions (-5°C, snowy) |
| QR camera unavailable | Manual URL entry + demo restaurant cards |
| URL has no allergen params | Shows dietary selector sheet, or proceeds with generic recommendations |
| Restaurant not in dataset | "Not found" screen with clear messaging |

---

## Real Data

Every restaurant in ClipCheck is real. The inspection records come from:

- **[Region of Waterloo Open Data](https://data.waterloo.ca/)** — Food inspection data for all Waterloo Region premises
- **[Toronto DineSafe](https://open.toronto.ca/dataset/dinesafe/)** — 15,000+ establishments, 130K+ inspection rows

Waterloo restaurants are **not required** to physically display inspection results. ClipCheck puts that data at the restaurant door.

---

## How the Trust Score Works

```
Score = weighted average of last 3 inspections

Weights:     Most recent: 60%  |  Second: 25%  |  Third: 15%
Penalties:   Crucial: -15  |  Significant: -8  |  Minor: -3
Adjustment:  +5 if trend is improving  |  -5 if declining

Result:      0-49 = Danger (red)  |  50-74 = Caution (amber)  |  75-100 = Safe (green)
```

---

## Team

Built at Hack Canada 2026.

---

## License

This project was built on the [Reactiv ClipKit Lab](https://github.com/reactivapp/reactivapp-clipkit-lab) framework. See the original repository for framework licensing.
