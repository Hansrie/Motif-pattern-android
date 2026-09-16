# Motif — Android APK

A thin native shell around the Motif web app. The entire app (HTML + CSS + JS)
sits in `app/src/main/assets/index.html` and runs fully offline inside a WebView.

## Two ways to get an APK

### A. GitHub Actions (no Android Studio needed)

1. Create a new GitHub repository.
2. Upload this whole folder to it (keep the folder structure exactly as is).
3. Open the **Actions** tab. The `Build APK` workflow runs automatically on push,
   or you can start it manually with **Run workflow**.
4. When it turns green, open the run and download the `motif-debug-apk` artifact.
5. Unzip it, transfer the `.apk` to your phone and install it.
   Android will ask you to allow installation from unknown sources — that is normal
   for an APK that did not come from the Play Store.

### B. Android Studio (local)

1. **File → Open** and select this folder.
2. Let Gradle sync (it downloads the Android Gradle Plugin on first run).
3. **Build → Build Bundle(s) / APK(s) → Build APK(s)**, or press Run with a
   phone connected over USB.

Output lands in `app/build/outputs/apk/debug/app-debug.apk`.

## What the native side adds

| Need | How it is handled |
|---|---|
| Icon upload (`<input type="file">`) | `WebChromeClient.onShowFileChooser` → system picker, multi-select enabled |
| Saving PNG / JPG / SVG exports | `window.MotifAndroid.saveFile()` → writes into `Downloads/Motif/` via MediaStore |
| Saved palettes, settings, theme | WebView DOM storage (`localStorage`) |
| Back button | Goes back in the WebView, then exits |

A WebView cannot download `blob:` or `data:` URLs on its own, which is why the
export path is bridged to Java instead of using a plain anchor download.

## Requirements and limits

- **minSdk 29** (Android 10). Uses scoped storage via MediaStore, so no storage
  permission is requested at all.
- **No dependencies** beyond the Android framework — no AndroidX, no Capacitor,
  no Cordova. The build is deliberately small.
- **Canvas size**: phone GPUs cap how large a canvas can be, typically around
  4096 × 4096. Exports above that can come out blank. The app shows a warning in
  the Canvas Size section when the requested size is risky. Full-size microstock
  exports (6000 × 6000) are best done in a desktop browser.
- Very large exports are passed to Java as a base64 string, which is memory
  hungry. Keep mobile exports at or below roughly 4000 × 4000.

## Updating the app

The web app is a single file. To ship a new version, replace
`app/src/main/assets/index.html` and bump `versionCode` / `versionName` in
`app/build.gradle`.

## Release (signed) build

The workflow produces a **debug** APK, which installs fine but is signed with a
throwaway debug key and cannot go on the Play Store. For a release build you
need to generate a keystore and add a `signingConfigs` block to
`app/build.gradle`.
