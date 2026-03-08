# Repository Guidelines

## Project Structure & Module Organization
Primary development happens in `reactiv_stuff/`.

- `reactiv_stuff/ReactivChallengeKit/ReactivChallengeKit/`: SwiftUI app source (simulator shell, router, protocols, components, examples).
- `reactiv_stuff/ReactivChallengeKit/ReactivChallengeKit/Submissions/`: compiled Clip submission code (this is the source of truth for app behavior).
- `reactiv_stuff/Submissions/`: duplicate/reference submission content used by scripts and docs; edits here alone do not change the running app.
- `reactiv_stuff/scripts/`: contributor scripts (`doctor.sh`, `create-submission.sh`, `generate-registry.sh`).
- `reactiv_stuff/docs/`: challenge and submission documentation.
- `.github/`: CI workflow and PR template.

## Build, Test, and Development Commands
Run commands from `reactiv_stuff/` unless noted.

- `bash scripts/doctor.sh`: environment preflight (Xcode/repo/template checks).
- `bash scripts/create-submission.sh "Team Name"`: scaffolds under `reactiv_stuff/Submissions/` (copy any files you intend to run into `ReactivChallengeKit/ReactivChallengeKit/Submissions/`).
- `bash scripts/generate-registry.sh`: regenerate `SubmissionRegistry.swift` from `Submissions/`.
- `xcodebuild -project ReactivChallengeKit/ReactivChallengeKit.xcodeproj -scheme ReactivChallengeKit -destination 'generic/platform=iOS Simulator' CODE_SIGNING_ALLOWED=NO build`: CI-equivalent build check.
- Xcode: open `reactiv_stuff/ReactivChallengeKit/ReactivChallengeKit.xcodeproj`, then `Cmd+R`.

## Coding Style & Naming Conventions
- Swift style: 4-space indentation, idiomatic SwiftUI composition, small focused views.
- Types/protocol conformers: `UpperCamelCase`; methods/properties: `lowerCamelCase`.
- Submission folders use kebab-case team slugs (for example, `team-42`).
- Clip experience structs should be descriptive and usually end with `ClipExperience`.
- URL patterns follow router syntax with `:param` segments (example: `example.com/store/:storeId/checkin`).
- Critical: edit compiled clip files in `ReactivChallengeKit/ReactivChallengeKit/Submissions/`, not only in `reactiv_stuff/Submissions/clipcheck/`.
- No enforced formatter/linter is configured; match surrounding style.

## Testing Guidelines
- No dedicated unit test target is currently configured; validation is build + runtime behavior.
- Before opening a PR, ensure:
  - project builds locally,
  - URL invocation works in `InvocationConsole`,
  - flow completes within the App Clip 30-second constraint.
- CI (`PR Validation - ClipKit Build`) regenerates registry and performs a simulator build.

## Commit & Pull Request Guidelines
- Recent history uses short imperative summaries, with occasional Conventional Commit prefixes (`feat:`). Prefer `type: concise summary` when possible (`feat: add QR scanner state handling`).
- Keep commits scoped to one logical change.
- Follow `.github/pull_request_template.md`:
  - include team/clip info,
  - complete checklist,
  - link demo video or screenshots,
  - avoid edits outside `Submissions/<team-slug>/` unless maintainers request otherwise.
