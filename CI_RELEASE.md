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
