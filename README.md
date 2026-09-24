# chat-center-native-version

Public version manifest for the B2Chat mobile app. Contains only version numbers — no secrets.

The app (`chat-center-native`, `src/shared/updates/`) fetches one of these files on launch and on
every return to the foreground, compares the numbers with its own native build number and decides
what to do: nothing, suggest a store update (a dismissable dialog), or block the app behind an
"update required" screen whose only button opens the store.

## Files

| File                   | Read by                                   | Who changes it                                                                      |
| ---------------------- | ----------------------------------------- | ----------------------------------------------------------------------------------- |
| `version.json`         | **production** builds (store customers)   | The `Update notices` workflow in `chat-center-native`. It reads what each store has live for every user and opens a PR here as the `b2chat-release-bot` GitHub App; merging it (one approval) is the publication. Form: platform (*both*, *android*, *ios*) and action — *Status*, *New version: optional update*, *New version: required update*, *Rollback: unblock everyone* |
| `version.preview.json` | **preview** builds (QA, `io.b2chat.appm.dev`) | By hand — the QA rehearsal knob. Never affects a store build.                   |

A build type whose file does not exist (development) gets a 404 and does nothing.

## Schema

```json
{
  "schemaVersion": 1,
  "ios":     { "buildNumber": 4,  "minBuildNumber": 4,  "updatedAt": "2026-09-22T00:00:00Z" },
  "android": { "buildNumber": 37, "minBuildNumber": 37, "updatedAt": "2026-09-22T00:00:00Z" }
}
```

Per platform (the counters are independent — EAS auto-increments them separately):

- `buildNumber` — the latest build **live in the store**. A device below it sees the "update
  available" dialog (Actualizar / Después, re-shown after 3 days).
- `minBuildNumber` — optional. The oldest build still allowed to run. A device below it is walled
  behind the "update required" screen. A build at or above `buildNumber` is never walled, so a
  `minBuildNumber` above `buildNumber` is harmless (treated as a typo).
- `updatedAt` — informative.

Numbers are compared numerically against the installed binary's build number
(`versionCode` on Android, `CFBundleVersion` on iOS).

## Rules

- **Publish `buildNumber` only once the store rollout is at 100 %** — a device that sees the
  dialog before the store offers the build gets sent to a listing with nothing to install.
- **Set `minBuildNumber` only for builds that can no longer receive OTA updates** (their native
  fingerprint is no longer the one on `main`) and only after the replacement has been live long
  enough for auto-updates to have done most of the work.
- **Rollback:** `Update notices` with *Rollback: unblock everyone* drops `minBuildNumber` (it
  works even when a store cannot be read); every walled app comes back on its next launch, no
  reinstall. Prefer the workflow over a hand edit: its PR shows each key before and after.
- Any failure to fetch or parse a file is silent: the app behaves as if the file said nothing.
- The app reads the GitHub contents API first (uncached, so a merge is visible on the next launch
  or foreground) and falls back to raw GitHub, which is CDN-cached for about 5 minutes — a
  cache-buster query does not defeat that cache.
