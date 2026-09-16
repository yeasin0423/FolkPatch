# AGENTS.md — FolkPatch (fork of APatch)

## Build
- Debug: `./gradlew assembleDebug` (output `app/build/outputs/apk/debug/`). Release: `./gradlew assembleRelease` (needs `keystore.properties`, see `keystore.properties.template`).
- Toolchain: Java 21, `minSdk 26 / target 36 / compile 37` (root `build.gradle.kts`). Linux helpers: `Build-Debug.sh`, `Build-Release.sh`.
- `preBuild` needs network + `cargo-ndk`: downloads `kpimg`/`kptools`/`*_kernelpatch.ko` into `app/src/main/assets`, builds `apd/` → `libs/arm64-v8a/libapd.so` and `fpd/` → `assets/Service/fpd`, merges `scripts/` into `META-INF/com/google/android`. First/offline builds fail without these. `mergeReleaseAssets` waits on `externalNativeBuild` (fpdrop).

## Structure
- Single module `:app` (`namespace me.bmax.apatch`, `applicationId me.yuki.folk`). Entrypoint `ui/MainActivity.kt` (Compose + compose-destinations `NavGraphs.root`); init in `APApplication`; Rust daemons `apd/` + `fpd/`; native `app/src/main/cpp/`.
- Launcher icon is 4 `activity-alias` entries (`MainActivityDefault/Alias/AliasSu/AliasAltSu`) toggled by `util/LauncherIconUtils.kt`. Real `MainActivity` has no `MAIN/LAUNCHER` filter — keep it that way.
- Splash: `Theme.Splash` (`res/values/themes.xml`) must keep `postSplashScreenTheme` + `windowSplashScreenBackground`/`windowSplashScreenAnimatedIcon`/`windowSplashScreenIconBackgroundColor` + legacy `android:windowBackground=@drawable/splash_background`. Background color is night-qualified (`values/colors.xml` + `values-night/colors.xml`). Dropping these reintroduces the Android 11 generic-APK-icon bug (fine on 12+ where the platform fills the launcher icon automatically).
- CI: `.github/workflows/build.yml` — debug needs no secrets (`workflow_dispatch` + `build_type: debug`); release needs `KEYSTORE_BASE64/PASSWORD/ALIAS/PASSWORD` secrets.
