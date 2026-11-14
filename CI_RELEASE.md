# Build Android in the cloud (no Android Studio needed)

The workflow `.github/workflows/android-build.yml` builds the app on GitHub's
servers. It runs on every push to this branch and can be triggered manually from
the **Actions** tab → *Build Android* → **Run workflow**.

## What you get

| Artifact | When | Use |
|---|---|---|
| `app-debug-apk` | always | Download, transfer to an Android phone, install to **test** the app. |
| `app-release-aab` | only if signing secrets are set | The **signed bundle to upload to Google Play**. |

Download artifacts from the bottom of each completed workflow run.

Each run also publishes the debug APK as a GitHub release, so it can be
downloaded straight from a phone (no login, no zip) at a link that never
changes:
https://github.com/Fetita/App-puntos-/releases/latest/download/anotador-truco.apk

> The debug APK is for testing only — Google Play requires the signed `.aab`.

## Enable the signed release build (one-time setup)

### 1. Create an upload keystore (on any machine with the JDK)
```bash
keytool -genkey -v -keystore truco-upload.jks \
  -keyalg RSA -keysize 2048 -validity 10000 -alias truco
```
Choose a strong password. **Back this file and password up somewhere safe** —
if you lose it you can't update the app on Play (unless you use Play App Signing,
which is recommended; see `LAUNCH_PLAN.md` §2.3).

### 2. Base64-encode the keystore
```bash
base64 -w0 truco-upload.jks > truco-upload.b64   # Linux
base64 -i truco-upload.jks   > truco-upload.b64   # macOS
```

### 3. Add four repository secrets
GitHub → repo **Settings → Secrets and variables → Actions → New repository secret**:

| Secret name | Value |
|---|---|
| `ANDROID_KEYSTORE_BASE64` | contents of `truco-upload.b64` |
| `ANDROID_KEYSTORE_PASSWORD` | the keystore password |
| `ANDROID_KEY_ALIAS` | `truco` (the alias from step 1) |
| `ANDROID_KEY_PASSWORD` | the key password (often same as keystore password) |

Once these exist, the next run produces `app-release-aab`. Upload that file in
Play Console → your app → **Production** (or a testing track) → **Create release**.

## Bumping the version for each release

Before each new Play release, increase the version in `android/app/build.gradle`:

```gradle
versionCode 2          // must increase by 1 every upload
versionName "1.1"      // the human-facing version
```

And bump the service-worker cache (`CACHE` in `www/sw.js`) so offline users get
the update.

---

# Build iOS in the cloud (no Mac needed)

The workflow `.github/workflows/ios-build.yml` builds the iOS app on a GitHub
macOS runner. It runs on every push that touches the app, and manually from
**Actions → Build iOS → Run workflow**.

> macOS runners count 10× against the free Actions minutes of private repos.

## What you get

| Artifact | When | Use |
|---|---|---|
| `ios-unsigned-ipa` (also published as release `ipa-<n>`) | always | Unsigned `.ipa`. Install it on an iPhone with **Sideloadly** or **AltStore**, which sign it with your own Apple ID (free accounts: the app expires after 7 days). |
| `ios-simulator-app` | always | `App.app` for the Xcode Simulator (on a Mac: drag it onto the simulator). |
| TestFlight build | manual run with **testflight** ticked + secrets set | Signed build uploaded straight to App Store Connect / TestFlight. |

## Enable TestFlight / App Store uploads (one-time setup)

Needs the **Apple Developer Program** (USD 99/year).

1. In https://appstoreconnect.apple.com → **Apps → +**, create the app with
   bundle id `com.anotadordetruco.app` (register it first in
   developer.apple.com → Identifiers if it doesn't show up).
2. **Users and Access → Integrations → App Store Connect API → Team Keys → +**.
   Give it the **Admin** role (Xcode needs it to create the signing
   certificate and profile automatically). Download the `.p8` (only once!) and
   note the **Key ID** and the **Issuer ID**.
3. Find your **Team ID** at developer.apple.com → Account → Membership.
4. Add these repository secrets (Settings → Secrets and variables → Actions):

| Secret name | Value |
|---|---|
| `APPSTORE_API_KEY_P8` | full text of the `.p8` file (including the BEGIN/END lines) |
| `APPSTORE_API_KEY_ID` | Key ID |
| `APPSTORE_API_ISSUER_ID` | Issuer ID |
| `APPLE_TEAM_ID` | Team ID |

5. **Actions → Build iOS → Run workflow**, tick **testflight**, run. The build
   number is the workflow run number, so it always goes up. A few minutes after
   it finishes the build shows up in App Store Connect → TestFlight.

To change the human-facing version, edit `MARKETING_VERSION` in
`ios/App/App.xcodeproj/project.pbxproj` (or in Xcode → target → General).
