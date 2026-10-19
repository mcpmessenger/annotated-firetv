# Amazon Fire TV Developer Hackathon — Product Feedback, Friction Log & Feature Requests

**Product:** Annotated for Fire TV (`com.annotated.app`)  
**Version:** `v1.0.1` — **Live on the Amazon Appstore** (Passed Primary, Content Policy, and Functionality Validations)  
**Target Device:** Fire TV Stick 4K Max 2nd Gen (Fire OS 8 / Android 11 / API 30)  
**Submission Category:** Fire TV Track  
**Evaluator Bonus:** Up to +10% Judging Bonus for Constructive Developer Feedback & Friction Log

---

## 1. Executive Summary

Building a high-performance 10-foot living room application with modern native-web hybrid technologies on Amazon Fire TV hardware proved remarkably viable. The Fire TV Stick 4K Max 2nd Gen provides impressive hardware-accelerated video decoding and responsive Chromium WebView runtimes.

However, developers targeting Fire OS face several non-obvious hurdles across remote input dispatching, media autoplay policies, hybrid manifest validation, and the Amazon Developer Console UX. Below is our structured **Friction Log** (containing specific task attempted, steps taken, expected vs. actual result, severity rating, workaround used, and actionable suggestion) followed by prioritized **Feature Requests** (`Critical` / `Important` / `Nice-to-have`).

---

## 2. Structured Friction Log

### Friction 1: WebView Media Autoplay & Unmuted TV Audio
* **Specific Task Attempted:** Auto-playing the first curated 1080p MP4 video clip with unmuted television audio immediately upon launching the Fire TV application.
* **Steps Taken:**
  1. Embedded an HTML5 `<video autoplay playsinline>` element inside the native Fire OS `WebView` container in `MainActivity.kt`.
  2. Sideloaded and launched the app on a physical Fire TV Stick 4K Max 2nd Gen connected to a 4K television.
* **Expected vs. Actual Result:**
  * **Expected:** Continuous, unmuted television playback upon cold start (matching standard broadcast/streaming TV behavior).
  * **Actual:** Video playback started muted or paused because Android Chromium `WebView` defaults to blocking unmuted media playback until a touchscreen user gesture is detected — even though Fire TV has no touchscreen.
* **Severity Rating:** **High** (Blocks core television lean-back UX on launch)
* **Workaround Used:** In `MainActivity.kt`, we explicitly configured `WebSettings`:
  ```kotlin
  settings.mediaPlaybackRequiresUserGesture = false
  settings.domStorageEnabled = true
  settings.javaScriptEnabled = true
  ```
  Additionally, we added an audio focus fallback listener in our JavaScript engine that listens for the very first D-pad remote `KeyEvent` (`handleTvKey`) to guarantee all media elements unmute at 100% volume.
* **Actionable Suggestion:** Document `settings.mediaPlaybackRequiresUserGesture = false` prominently in the Amazon Fire TV Web App starter documentation, or provide an official `FireTvWebView` component that defaults this setting to `false` on Fire OS TV devices.

---

### Friction 2: D-Pad Keycode Mapping, Focus Traps & Dual Enter Codes
* **Specific Task Attempted:** Navigating our 7-action bottom reaction dock (`Annotate`, `Fire`, `Think`, `Idea`, `100`, `Down`, `Dispute/Report`) and modal dialogs using the physical Fire TV remote.
* **Steps Taken:**
  1. Attached standard DOM `keydown` listeners inside the web UI and tested navigation with the Alexa Voice Remote and HDMI-CEC television remotes.
  2. Navigated across active `<video>` players and floating HUD overlays.
* **Expected vs. Actual Result:**
  * **Expected:** Deterministic Left/Right/Up/Down navigation and consistent `Enter` selection across all remote types.
  * **Actual:** Chromium's built-in spatial navigation created focus traps inside HTML `<video>` media controls, and different remotes emitted either `KEYCODE_DPAD_CENTER` (keycode 23) or `KEYCODE_ENTER` (keycode 66), causing missed clicks.
* **Severity Rating:** **High** (Breaks remote navigation determinism)
* **Workaround Used:** We intercepted all hardware key events at the native Kotlin shell level before the `WebView` could swallow them:
  ```kotlin
  override fun dispatchKeyEvent(event: KeyEvent): Boolean {
      if (event.action == KeyEvent.ACTION_DOWN) {
          when (event.keyCode) {
              KeyEvent.KEYCODE_DPAD_UP -> bridge("ArrowUp")
              KeyEvent.KEYCODE_DPAD_DOWN -> bridge("ArrowDown")
              KeyEvent.KEYCODE_DPAD_LEFT -> bridge("ArrowLeft")
              KeyEvent.KEYCODE_DPAD_RIGHT -> bridge("ArrowRight")
              KeyEvent.KEYCODE_DPAD_CENTER, KeyEvent.KEYCODE_ENTER -> bridge("Enter")
              KeyEvent.KEYCODE_BACK -> bridgeBack()
          }
      }
      return super.dispatchKeyEvent(event)
  }
  ```
* **Actionable Suggestion:** Publish an official lightweight NPM package (e.g., `@amazon/firetv-web-nav`) and Kotlin snippet with standardized 10-foot remote key normalization (`DPAD_CENTER` + `ENTER`), focus ring CSS primitives, and spatial focus trap prevention.

---

### Friction 3: Hybrid Device Manifests (`android.software.leanback`) in Amazon Appstore
* **Specific Task Attempted:** Configuring `AndroidManifest.xml` to pass Fire TV Appstore validation while maintaining broad compatibility across Fire OS devices.
* **Steps Taken:**
  1. Declared `<uses-feature android:name="android.software.leanback" android:required="true" />` in `AndroidManifest.xml`.
  2. Uploaded the signed release APK to the Amazon Developer Console and inspected the device support matrix.
* **Expected vs. Actual Result:**
  * **Expected:** Seamless targeting across the Fire OS ecosystem.
  * **Actual:** Setting `android:required="true"` caused Amazon's ingestion parser to strictly exclude non-leanback Fire tablets, while omitting leanback flags entirely risked failing TV launcher validation.
* **Severity Rating:** **Medium**
* **Workaround Used:** We tuned `AndroidManifest.xml` to declare both features as optional (`required="false"`) while registering dual intent filters on `MainActivity`:
  ```xml
  <uses-feature android:name="android.software.leanback" android:required="false" />
  <uses-feature android:name="android.hardware.touchscreen" android:required="false" />
  ```
  ```xml
  <category android:name="android.intent.category.LAUNCHER" />
  <category android:name="android.intent.category.LEANBACK_LAUNCHER" />
  ```
* **Actionable Suggestion:** Add an interactive manifest preview validator in the Amazon Developer Console that clearly explains how `android.software.leanback` and `LEANBACK_LAUNCHER` affect device eligibility prior to submission.

---

### Friction 4: Developer Console Resetting DRM & Device Support on APK Version Updates
* **Specific Task Attempted:** Uploading the `v1.0.1` release APK update after `v1.0.0` in the Amazon Developer Console.
* **Steps Taken:**
  1. Built and signed `app-release.apk` (`versionCode 2`, `versionName "1.0.1"`).
  2. Uploaded the APK under the new version tab in the Amazon Developer Console.
* **Expected vs. Actual Result:**
  * **Expected:** DRM preferences and supported Fire TV device selections from `v1.0.0` would automatically carry over to `v1.0.1`.
  * **Actual:**
    1. The DRM selection radio button defaulted to unselected, throwing an inline validation error.
    2. The console displayed a blocking warning: *"This APK currently supports no devices. In order to submit the app, this APK must support at least one device."*
* **Severity Rating:** **Medium** (Confusing UX blocker during version updates)
* **Workaround Used:** Manually re-selected DRM preferences, clicked into the *"Edit device support"* modal, and toggled all supported Fire TV devices back on before submitting `v1.0.1` (which subsequently passed all three validations: Primary, Content Policy, and Functionality).
* **Actionable Suggestion:** Automatically inherit DRM preferences and device targeting from the previous APK version when uploading a patch binary, or provide a single-click *"Copy settings from previous version"* button.

---

### Friction 5: Wi-Fi ADB Connection Drops When Fire TV Stick Enters Standby
* **Specific Task Attempted:** Iterative wireless debugging and sideloading (`gradlew installDebug`) over Wi-Fi ADB (`192.168.12.188:5555`).
* **Steps Taken:**
  1. Connected to the Fire TV Stick over Wi-Fi via `adb connect 192.168.12.188:5555`.
  2. Wrote code for 10–15 minutes while the TV screen sat idle, then ran `./gradlew installDebug`.
* **Expected vs. Actual Result:**
  * **Expected:** ADB connection remains active on the local network or wakes the device on deploy.
  * **Actual:** Fire OS puts the wireless ADB daemon to sleep during standby, causing Gradle builds to fail with `no devices/emulators found` or `unauthorized` until reconnected.
* **Severity Rating:** **Low / Medium**
* **Workaround Used:** Scripted our PowerShell build pipeline to explicitly reconnect and dispatch `KEYCODE_WAKEUP` (`224`) prior to installing builds:
  ```powershell
  adb connect 192.168.12.188:5555
  adb shell input keyevent 224 # KEYCODE_WAKEUP
  ./gradlew installDebug
  ```
* **Actionable Suggestion:** Add a toggle under **Settings → My Fire TV → Developer Options**: *"Keep ADB over Network active during sleep"*.

---

## 3. Prioritized Feature Requests for the Fire TV Platform

### Feature Request 1: Official `FireTvWebView` / Leanback Web Component
* **Priority Rating:** **Critical**
* **Description:** An official AndroidX / Fire OS `FireTvWebView` wrapper pre-configured for 10-foot living-room applications (unmuted media autoplay enabled by default, hardware layer acceleration enforced, and normalized D-pad `KeyEvent` bridging).
* **Why It Matters to Our Project:** Annotated streams continuous HD video clips with real-time HUD overlays. Eliminating boilerplate WebView media workarounds and spatial focus traps would save every hybrid Fire TV developer days of debugging.

### Feature Request 2: Preserve DRM & Device Targeting Across Version Uploads in Developer Console
* **Priority Rating:** **Important**
* **Description:** Carry over DRM selections and device support filters automatically when uploading a new APK version in the Amazon Appstore Developer Console.
* **Why It Matters to Our Project:** When shipping rapid updates (`v1.0.0` → `v1.0.1`), resetting device support to zero devices creates unnecessary friction and risk of submission errors.

### Feature Request 3: Fire TV Second-Screen QR / WebSocket Companion SDK
* **Priority Rating:** **Important**
* **Description:** A lightweight Amazon SDK that generates pairing QR codes on Fire TV and establishes a local WebSocket/WebRTC relay with the viewer's smartphone browser (without requiring a separate mobile app install).
* **Why It Matters to Our Project:** Typing fact-check citations or annotations on an on-screen TV keyboard is slow. Annotated built a custom QR-to-Phone bridge for second-screen annotations and disputes; making this a first-party Fire TV SDK primitive would unlock rich social and multi-player living-room apps.

### Feature Request 4: Native 10-Foot TV Safe-Area CSS Environment Variables
* **Priority Rating:** **Nice-to-have**
* **Description:** Expose `env(safe-area-inset-top/right/bottom/left)` inside Fire OS `WebView` calibrated to television HDMI overscan boundaries (standard 5% / 48px vertical, 96px horizontal action-safe margins).
* **Why It Matters to Our Project:** Ensuring our bottom 7-action reaction dock and top-right QR dispute cards never clip on older consumer televisions required manual viewport padding calibration.
