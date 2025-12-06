# GitHub Copilot Instructions – Mobile Repository

This extends the global instructions from the [specs repository](https://github.com/localstore-platform/specs/blob/v1.1-specs/.github/copilot-instructions.md).

## Repository Context

- **Repository:** mobile
- **Purpose:** Flutter mobile app for restaurant owners to manage their business
- **Tech Stack:** Flutter 3.x, Dart 3.x, Riverpod 2.4, Dio, Hive, Firebase
- **Status:** 🟡 Planned for Sprint 1 (after Sprint 0.5 Menu Demo)
- **Spec Version:** v1.1-specs

---

## 🚀 "Continue Work" Trigger

When the user says **"continue work"**, **"tiếp tục"**, or **"next task"**:

### Step 1: Read Local Progress

Read `docs/CURRENT_WORK.md` in this repository to understand:

- Current sprint and focus
- Which stories are assigned to this repo
- Current status of each story (🔴 Not Started / 🟡 In Progress / ✅ Done)
- Any blockers or notes from previous session

### Step 2: Identify Next Task

From CURRENT_WORK.md:

1. If any story is 🟡 In Progress → continue that story
2. Otherwise, pick the first 🔴 Not Started story
3. If all stories are ✅ Done → report completion and suggest next sprint

### Step 3: Load Specifications

1. Check the "Spec References" section in CURRENT_WORK.md for the spec link
2. Fetch and read the relevant specification section
3. Primary spec: `flutter-mobile-app-spec.md` (entire file)

### Step 4: Implement

1. Follow the spec exactly
2. Apply code standards from this file (see below)
3. Clean Architecture, Riverpod state management

### Step 5: Update Progress

After implementation, **update `docs/CURRENT_WORK.md`**:

- Change story status from 🔴 to 🟡 (in progress) or ✅ (done)
- Add notes about what was implemented
- Add any blockers or follow-up items
- Update "Last Updated" timestamp

### Step 6: Git Workflow

1. Create/use feature branch: `feat/<story-description>`
2. Commit with conventional message
3. Run `flutter analyze && flutter test`
4. Create PR when story is complete

### Step 7: Post Event to Slack

After completing a story, post an event to `#agent-events` (channel ID: `C0A1VSFQ9SS`):

```
📱 STORY_DONE from mobile: [Story Title]
---
Details: [What screens/features were implemented]
Affected: [api, specs]
Action: [Any API needs or spec updates required]
Spec: [Link to relevant spec section]
```

Use `slack_post_message` with channel_id `C0A1VSFQ9SS`.

### Step 8: Report

Tell the user:

- What story you worked on
- What was implemented
- Current status
- What's next
- Event posted to Slack

---

## 🔄 "Sync Events" Trigger

When the user says **"sync events"**, **"check events"**, or **"đồng bộ"**:

1. **Read recent messages** from `#agent-events` (channel ID: `C0A1VSFQ9SS`) using `slack_get_channel_history`
2. **Filter for events affecting this repo** (look for "mobile" in Affected field)
3. **Report relevant events** to user with recommended actions
4. **If API_READY from api** → integrate new endpoints
5. **If SCHEMA_UPDATED from contracts** → update Dart models to match

---

## Key Spec Files

| Spec File | Sections | Purpose |
|-----------|----------|---------|
| `architecture/flutter-mobile-app-spec.md` | Entire file | Complete app specification |
| `architecture/flutter-mobile-app-spec.md` | Lines 90-470 | Project structure |
| `architecture/flutter-mobile-app-spec.md` | Lines 205-800 | Feature screens |
| `architecture/flutter-mobile-app-spec.md` | Lines 810-1105 | Design system |
| `design/wireframes-ux-flow.md` | Lines 400-800 | Mobile owner dashboard |

Also check `docs/SPEC_LINKS.md` for curated links with line numbers.

---

## Build & Test Commands

```bash
flutter pub get          # Install dependencies
flutter run              # Run on device/emulator
flutter test             # Run tests
dart analyze             # Analyze code
dart format .            # Format code
flutter build apk --release   # Build Android APK
flutter build ios --release   # Build iOS (requires macOS)
dart run build_runner build   # Generate code (Riverpod, JSON)
```

---

## Project Structure

```text
lib/
├── core/
│   ├── theme/            # AppColors, Typography, Spacing
│   ├── utils/            # Formatters (VND, dates)
│   └── network/          # Dio client, interceptors
├── features/
│   ├── auth/             # Phone OTP login
│   ├── dashboard/        # Metrics, trends
│   ├── recommendations/  # AI suggestions
│   └── notifications/    # Push notification list
├── shared/
│   ├── widgets/          # AppButton, PhoneInput, EmptyState
│   └── providers/        # Global Riverpod providers
└── main.dart
```

---

## Code Standards

### Clean Architecture

- **Presentation:** Widgets, pages, state (Riverpod)
- **Domain:** Entities, use cases, repository interfaces
- **Data:** Repository implementations, data sources, DTOs

### Riverpod Patterns

- Use `@riverpod` annotation with code generation
- `AsyncValue` for loading/error/data states
- Family providers for parameterized data

### Vietnamese Localization

- Default locale: vi-VN
- Currency: VND with format 75.000₫
- All user-facing text in Vietnamese

---

## Git Workflow

**CRITICAL:** Follow `docs/GIT_WORKFLOW.md`:

### Never Commit Directly to Main

- **Always** create a new branch for changes
- If currently on `main`, checkout to a new branch before committing
- Branch naming: `<type>/<short-description>`

### Branch Naming Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `chore` | Maintenance, dependencies, tooling |
| `docs` | Documentation only |
| `refactor` | Code refactoring |
| `test` | Adding or updating tests |

### Logical Commits

- Group related changes into logical commits
- Each commit should represent a single logical change
- Use conventional commit format:

```text
<type>: <short description>

- Detail 1
- Detail 2
```

### Commit Granularity Principle

Each commit should answer ONE question: "What single feature/fix does this add?"

**Rule of thumb:** If you need "and" to describe the commit, split it.

- ❌ `Add docs and config files` → Split
- ✅ `Add GitHub PR template and CODEOWNERS` → OK (same purpose: GitHub config)

### Pull Request Workflow

After committing:

1. Push the branch to origin
2. Create a PR to `main` branch
3. If PR already exists, update the title and description
4. **Do not wait for confirmation** - push and create PR automatically

---

## Dependencies

- **Requires:** `api` repo for REST/GraphQL endpoints
- **Uses:** `contracts` repo types as reference (not direct import in Dart)
- **Firebase:** For push notifications

---

## Related Documentation

- [Flutter Mobile App Spec](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/flutter-mobile-app-spec.md)
- [Wireframes & UX Flow](https://github.com/localstore-platform/specs/blob/v1.1-specs/design/wireframes-ux-flow.md)
- [API Specification](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/api-specification.md)
