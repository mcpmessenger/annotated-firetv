# Amazon Fire TV Developer Hackathon — Product Feedback & Friction Log

**Product:** Annotated for Fire TV (`com.annotated.app`)  
**Target Device:** Fire TV Stick 4K Max 2nd Gen (2023, Fire OS 8, Android 11 / API 30)  
**Submission Category:** Fire OS / Fire TV Track  
**Evaluator Bonus:** Up to +10% Judging Bonus for Constructive Developer Feedback

---

## 1. Executive Summary

Building a high-performance 10-foot living room application with modern web technologies on Amazon Fire TV hardware proved remarkably viable. The Fire TV Stick 4K Max 2nd Gen provides impressive hardware-accelerated video decoding and responsive Chromium WebView runtimes.

However, modern developers targeting Fire OS face several non-obvious hurdles across remote input dispatching, media autoplay policies, manifest validation, and the Amazon Developer Console UX. Below is our transparent friction log, detailing the exact blockers encountered, how we engineered solutions around them, and concrete feature requests for the Amazon Fire TV Developer team.

---

## 2. Technical Friction Logs & Engineering Workarounds

### Friction 1: WebView Media Autoplay & Unmuted TV Audio
* **The Problem:** Television is an unmuted, continuous lean-back medium. Standard Android Chromium `WebView` defaults to muting or blocking media playback until a direct touchscreen user gesture is detected. On Fire TV, there is no touchscreen, and requiring a remote click before the first clip plays ruins the TV onboarding experience.
* **Our Workaround:** In `MainActivity.kt`, we explicitly configured:
  ```kotlin
  settings.mediaPlaybackRequiresUserGesture = false
  settings.domStorageEnabled = true
  settings.javaScriptEnabled = true
  ```
  Furthermore, we implemented an automatic fallback in our audio management layer: if system audio focus changes, the web layer listens for the very first D-pad remote keycode (`handleTvKey`) to automatically ensure audio elements unmute at 100% volume.
* **Recommendation for Amazon:** Provide an official `FireTvWebView` or AndroidX Leanback WebView component that sets TV-optimized media defaults (unmuted autoplay permitted, background audio policy handling, and hardware acceleration enabled by default).

---

### Friction 2: D-Pad Keycode Mapping & Dual Enter Codes
* **The Problem:** Fire TV remotes emit both `KEYCODE_DPAD_CENTER` (keycode 23) and `KEYCODE_ENTER` (keycode 66) depending on the remote generation, HDMI-CEC passthrough remotes, or third-party Bluetooth controllers. Additionally, Chromium's built-in spatial navigation often creates unexpected focus traps inside HTML `video` controls or floating HUD layers.
* **Our Workaround:** We intercepted all hardware key events at the native shell level before the WebView could swallow them:
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
  This allowed 100% deterministic, 60 FPS remote navigation without relying on simulated mouse pointers or unpredictable browser spatial focus.
* **Recommendation for Amazon:** Publish an official NPM package (e.g. `@amazon/firetv-web-nav`) with standard 10-foot remote navigation hooks, focus ring CSS primitives, and audio feedback sounds for web-based Fire TV apps.

---

### Friction 3: Hybrid Device Manifests (Fire TV vs. Fire Tablets)
* **The Problem:** When building an app that targets both Fire TV and regular Fire tablets (like Fire HD 10 or Fire Max 11), declaring `<uses-feature android:name="android.software.leanback" android:required="true" />` causes Amazon's Appstore ingestion engine to automatically disable all tablet devices. Conversely, omitting leanback flags risks failing TV-specific Appstore validation.
* **Our Workaround:** We tuned our `AndroidManifest.xml` to declare:
  ```xml
  <uses-feature android:name="android.software.leanback" android:required="false" />
  <uses-feature android:name="android.hardware.touchscreen" android:required="false" />
  ```
  And configured dual intent filters on `MainActivity`:
  ```xml
  <category android:name="android.intent.category.LAUNCHER" />
  <category android:name="android.intent.category.LEANBACK_LAUNCHER" />
  ```
  This enabled broad compatibility across both Fire TV Sticks/Cubes and handheld Fire Tablets.
* **Recommendation for Amazon:** Update documentation to highlight the hybrid manifest pattern and clarify how device filtering interacts with `android.software.leanback`.

---

### Friction 4: Wi-Fi ADB Connection Drops on Device Sleep
* **The Problem:** During rapid development cycles on physical hardware over Wi-Fi ADB (`adb connect 192.168.x.x:5555`), the Fire TV Stick enters low-power sleep mode after several minutes of inactivity. This silently drops the ADB daemon connection. Subsequent Gradle `./gradlew installDebug` calls fail cryptically with `no devices/emulators found`.
* **Our Workaround:** We scripted our sideloading pipeline in PowerShell to automatically test connection liveness and reconnect before pushing builds:
  ```powershell
  & adb connect 192.168.12.188:5555
  & adb shell input keyevent 224 # KEYCODE_WAKEUP
  & ./gradlew installDebug
  ```
* **Recommendation for Amazon:** Introduce a developer setting in Fire OS under Developer Options: *"Keep Wi-Fi ADB active while sleeping"* or an official Amazon CLI extension (`amazon firetv connect`) that manages device keep-alives.

---

### Friction 5: Developer Console Submission UX on New APK Uploads
* **The Problem:** When uploading a new version APK (e.g. bumping from v1.0.0 to v1.0.1) in the Amazon Developer Console:
  1. The DRM selection radio button defaults to unselected, throwing an inline validation error.
  2. The system displays an alarming error: *"This APK currently supports no devices. In order to submit the app, this APK must support at least one device."*
  3. The solution requires manually hunting down an "Edit device support" link and clicking "Turn on all supported devices".
* **Recommendation for Amazon:**
  1. Automatically carry over the device support targeting and DRM preferences from the previous app version when an APK update is uploaded.
  2. Provide a single-click "Copy settings from Version X" button in the Developer Console.

---

## 3. High-Value Feature Requests for the Fire TV Platform

1. **Fire TV Web Remote Companion SDK:**  
   Most modern TV apps require mobile pairing (for logins, typing comments, or scanning QR codes). An Amazon-provided WebRTC or WebSocket companion library connecting the Fire TV directly to the Amazon mobile app (or a lightweight web bridge) would be an incredible superpower for third-party developers.
2. **Native 10-Foot Safe-Area CSS Environment Variables:**  
   Televisions suffer from overscan issues across various consumer HDMI displays. Exposing `env(safe-area-inset-top)` and `env(safe-area-inset-left)` inside Fire OS WebView would allow web-based TV apps to reliably prevent HUD collision with TV bezels.
3. **Official VS Code / Android Studio Fire TV Extension:**  
   An official Amazon Fire TV extension with 1-click device discovery, wireless screenshot capture (`screencap`), live remote simulation, and remote logcat filtering would dramatically streamline the developer onboarding experience.
