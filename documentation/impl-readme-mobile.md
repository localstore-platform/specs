README Template — local-store-mobile

Purpose
Mobile app repository (Flutter) for owner dashboard and staff apps.

Quick summary

- Tech: Flutter, Dart
- Audience: Mobile engineers, QA, product designers
- Responsibility: Mobile UI, push notifications, offline-first caching

Suggested layout

- /lib/
- /assets/
- /android/ /ios/
- /integration_tests/

Onboarding

1. Install Flutter stable channel
2. flutter pub get
3. flutter run

CI & Releases

- CI: run unit tests, widget tests, and integration tests in emulators
- Release: use Fastlane for iOS/App Store and Android Play Console automation

Contracts

- Use GraphQL or REST endpoints from `local-store-contracts` depending on chosen client strategy.

Ownership

- CODEOWNERS: mobile lead
- Contacts: mobile@ourplatform

Notes

- Aim for small APK/IPA sizes; offline support for low-bandwidth environments.
