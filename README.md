# FlatRadar for Android

[![Android Tests](https://img.shields.io/github/actions/workflow/status/751K/FlatRadar-Android/android.yml?style=flat-square&label=tests)](https://github.com/751K/FlatRadar-Android/actions/workflows/android.yml)
[![Platform](https://img.shields.io/badge/Android-8.0%2B-3DDC84?style=flat-square&logo=android&logoColor=white)](https://developer.android.com/)
[![Compose](https://img.shields.io/badge/Jetpack_Compose-Kotlin-7F52FF?style=flat-square&logo=kotlin&logoColor=white)](https://developer.android.com/jetpack/compose)
[![License](https://img.shields.io/badge/license-PolyForm%20Noncommercial%201.0.0-blue?style=flat-square)](https://github.com/751K/Flatradar/blob/master/LICENSE)

The Android client for [FlatRadar](https://flatradar.app), a monitor for the
Dutch rental market. It watches seven platforms — Holland2Stay, OurDomain,
OurCampus, Xior, Magis, Plaza and Student Experience — and pushes a notification
the moment a listing matches what you are looking for.

The scraping, matching and notification pipeline lives in the backend repo,
[751K/Flatradar](https://github.com/751K/Flatradar). This
repo is the app that sits in front of it. The iOS client is at
[751K/FlatRadar-iOS](https://github.com/751K/FlatRadar-iOS).

## What it does

- **Dashboard** — supply and price trends across the platforms you follow.
- **Listings, map and calendar** — browse what is available and when.
- **Filters** — city, price, rooms, contract, energy label, furnishing and more.
- **Alerts** — FCM push for new matches, plus a live in-app feed over SSE.
- **Biometric sign-in**, guest browsing, and admin tools for self-hosters.

English and 简体中文.

## Requirements

- Android 8.0+ (API 26)
- JDK 17, Android SDK 35

## Build

```bash
git clone https://github.com/751K/FlatRadar-Android.git
cd FlatRadar-Android
./gradlew :app:assembleDebug
```

Two files are **not** in the repo and you need to supply them yourself:

| File | What it is |
|---|---|
| `app/google-services.json` | Firebase config, for FCM push. Download it from the Firebase console. |
| `local.properties` | `sdk.dir` plus `MAPS_API_KEY=…` for the map screen. |

Without `google-services.json` the Gradle configuration phase fails — the Google
Services plugin wants it before any task is generated, so this is not a runtime
error you can ignore. CI writes a structurally valid placeholder for the test job
so that unit tests do not need a secret (and therefore still run on pull requests
from forks).

## Layout

```text
app/src/main/java/com/flatradar/app/
├── data/local/      TokenManager, PreferencesManager, BiometricAuth
├── data/remote/     ApiClient, ApiService, AuthInterceptor, SseClient
├── domain/model/    API models
├── di/              Hilt modules
├── navigation/      NavHost, coordinator, top-level destinations
├── push/            FCM service, token registration, channels
├── ui/              Compose screens + ViewModels
└── util/            CrashReporter, ServerTime
app/src/test/        Unit tests (ViewModels)
docs/ANDROID_PLAN.md Roadmap and platform notes
```

## Testing

```bash
./gradlew :app:testDebugUnitTest
```

`android.yml` runs these on every push — and then **counts the test cases in the
XML report**. Before the split these five test files had never run in CI at all:
the workflow only built APKs. A suite nobody runs looks exactly like a suite that
passes, which is why the count is asserted rather than trusted.

## CI

| Workflow | Trigger | What it does |
|---|---|---|
| `android.yml` | every push / PR | unit tests, and asserts they actually ran |
| `release.yml` | `v*` tag / manual | signed APK, verified with `apksigner`, attached to the Release |

Distribution is the Release page — a direct APK download. Google Play was
considered and dropped (2026-08-03), so no AAB is produced.

## The contract with the backend

One file connects this repo to the backend: `docs/openapi.json` over there. The
backend derives its route set from the live Flask app and diffs it against that
spec in both directions, so the spec fails loudly when it falls behind. Treat it
as the source of truth for anything over the wire.

## Security

The repo is public and holds no credentials. `android.yml` needs none. Release
signing reads GitHub Secrets (`ANDROID_KEYSTORE_BASE64`, `ANDROID_KEY_ALIAS`,
`ANDROID_KEY_PASSWORD`, `ANDROID_STORE_PASSWORD`, `GOOGLE_SERVICES_JSON_BASE64`,
`MAPS_API_KEY`); nothing else does.
