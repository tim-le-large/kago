# KaGo — Karlsruhe transit (KVV)

**Production-shaped** Flutter client for the Karlsruhe region: search stops, browse departures, plan connections. A **BLoC + repository** reference with a thin **Cloudflare Worker** that translates TRIAS XML → JSON.

| | |
|---|---|
| **Live demo** | [kago.lelarge.dev](https://kago.lelarge.dev) |
| **Portfolio** | [lelarge.dev](https://lelarge.dev) |
| **API** | [api.kago.lelar.ge](https://api.kago.lelar.ge) |
| **Author** | [Tim Le Large](https://github.com/tim-le-large) |

---

## What this demo shows

- **API integration:** Real HTTP against a custom backend — not hardcoded JSON in the app
- **Clean architecture:** Feature slices (`data` / `bloc` / `presentation`) with testable repositories
- **State management:** BLoC with `bloc_concurrency` restartable transformers for search
- **Thin edge backend:** Cloudflare Worker holds TRIAS credentials; Flutter only sees public JSON

Good reference for **domain apps with external APIs** — ÖPNV, logistics, or any REST-shaped integration.

---

## Features

| Area | Details |
|------|---------|
| **Stops** | Debounced search, list → detail navigation |
| **Departures** | Chronological board, pull-to-refresh, periodic UI refresh |
| **Trips** | Origin/destination search and connection results |
| **Settings** | Light / dark theme (persisted via `shared_preferences`) |
| **Offline dev** | Optional fake repositories (`USE_FAKE_LOCATIONS=true`) |

---

## Architecture

```
lib/
  config/           # API base URL, compile-time toggles
  shell/            # Bottom navigation
  locations/        # data · bloc · presentation
  departures/       # data · bloc · presentation
  trips/            # data · bloc · presentation
  settings/         # data · bloc · presentation
  theme/            # Material 3 light/dark
  widgets/          # Line badges, empty states
api-proxy/          # Cloudflare Worker (TRIAS → JSON)
```

**Data flow:** UI → BLoC/Cubit → `Http*Repository` → Worker JSON API. Departures use a route-scoped `BlocProvider` so subscriptions do not leak across navigation.

---

## API

The app talks only to JSON endpoints (never parses TRIAS XML).

Default base URL: `lib/config/api_config.dart` → `https://api.kago.lelar.ge`

| Method | Path | Purpose |
|--------|------|---------|
| `GET` | `/api/v1/locations?q=…` | Stop search |
| `GET` | `/api/v1/departures?stopRef=…` | Departures at a stop |
| `POST` | `/api/v1/trips` | Connection search |

Worker details: [`api-proxy/README.md`](api-proxy/README.md)

---

## Quick start

### 1. API proxy (optional)

```bash
cd api-proxy
npm install
export TRIAS_ENDPOINT=https://projekte.kvv-efa.de/lelargetrias/trias
export TRIAS_REQUESTOR_REF=your-ref
npm run dev
```

Runs at `http://127.0.0.1:8787` — point `ApiConfig` at this URL for local dev.

### 2. Flutter app

```bash
flutter pub get
flutter run -d chrome
```

**UI-only mode** (no backend):

```bash
flutter run --dart-define=USE_FAKE_LOCATIONS=true
```

---

## Tests

```bash
flutter test
```

| File | Covers |
|------|--------|
| `test/departure_test.dart` | `Departure.fromJson`, time parsing |
| `test/location_test.dart` | `Location.fromJson` |
| `test/trip_model_test.dart` | `Leg.isWalk`, `Journey.fromJson` |
| `test/widget_test.dart` | App boot, tab navigation |

---

## Security

Do **not** commit `TRIAS_REQUESTOR_REF` or embed it in the Flutter app. The Worker holds the secret; the client only sees public JSON.

---

## Deploy

- **Web:** GitHub Actions → GitHub Pages (see `.github/workflows/deploy-frontend-pages.yml`)
- **API:** `cd api-proxy && npx wrangler deploy`

---

## Related portfolio projects

| App | Stack | Live |
|-----|-------|------|
| [SquadBoard](https://github.com/tim-le-large/squadboard) | Firebase, Riverpod | [squadboard.lelarge.dev](https://squadboard.lelarge.dev) |
| [FinFlow](https://github.com/tim-le-large/finflow) | Supabase, Riverpod | [finflow.lelarge.dev](https://finflow.lelarge.dev) |

---

## License

MIT — see [LICENSE](LICENSE).
