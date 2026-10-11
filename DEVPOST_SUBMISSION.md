# Devpost Submission — Annotated for Fire TV

**Submission Title:** Annotated for Fire TV  
**Tagline:** Your personal TV channel, not the algorithm’s — watch it together, and the TV talks back.  
**Track:** Amazon Fire OS / Fire TV Track  
**Priority Categories Addressed:** Family Entertainment · AI-Enhanced Viewing · Multi-Modal UX  
**GitHub Repository:** [mcpmessenger/annotated-firetv](https://github.com/mcpmessenger/annotated-firetv) (MIT License, Public)  
**Package:** `com.annotated.app`  
**Target Hardware:** Amazon Fire TV Stick 4K Max 2nd Gen (Fire OS 8 / Android 11)

---

## 💡 Inspiration: The Anti-Algorithm TV Channel

Traditional television is a passive, one-way broadcast. Modern streaming apps like TikTok or YouTube force you into algorithmic echo chambers where the platform decides what you consume, and viewers have zero context regarding whether the claims made on screen are factual, satirical, or debunked.

Meanwhile, in the living room, multiple people sit together on the couch — each with a smartphone in hand, completely disconnected from the broadcast on the big screen.

We asked: **What if your television played a personal channel of content YOU and your family curated, and the TV actually talked back?**

**Annotated for Fire TV** transforms television into an interactive, communal experience:
1. **Your channel, not the algorithm’s:** It plays clips, posts, and articles you saved and highlighted across the web.
2. **Watch it together:** The whole living room reacts in real-time using the physical Fire TV remote.
3. **The TV talks back:** Every claim carries live community context, AI fact-checking notes, and instant smartphone dispute passes.

---

## 📺 What It Does

Annotated for Fire TV is a native 10-foot lean-back client engineered specifically for Amazon Fire TV hardware. It plays a continuous, full-screen stream of video clips, Twitter/X threads, and web articles with a rich, glassmorphic HUD overlay:

### 1. Interleaved Multi-Format Feed Engine
The app auto-advances through curated video clips (`.mp4`), tweets, and long-form web articles in an interleaved sequence (`Video → Tweet → Article → Video`). Videos play with full unmuted TV audio up to 90 seconds; static posts display for 15 seconds with a glowing bottom countdown timer. Viewers can surf forward and backward through items using the remote’s **D-pad Up / Down** buttons.

### 2. Ambient AI Truth HUD (At-a-Glance Verification)
Viewers absorb context without reading dense text panels:
* 🟢 **Green Accent:** Verified Accurate (backed by primary sources & positive community consensus).
* 🟡 **Yellow Accent:** Needs Context (satire/comedy, opinion, or unverified claims).
* 🔴 **Red Accent:** Disputed / Debunked (stamped with a prominent diagonal "FALSE" rubber stamp overlay).

### 3. Remote Control Reaction Dock (7 Actions)
Using **D-pad Left / Right** and **OK / Select**, anyone on the couch can react immediately:
* `✏️ Annotate` — Opens the QR phone bridge to contribute a timestamped note.
* `🔥 Fire` — Up-vote / endorse.
* `🤔 Think` — Mark as thought-provoking.
* `💡 Idea` — Mark as insightful.
* `💯 100` — Strong consensus agreement.
* `👎 Down` — Down-vote / dispute signal.
* `⚠️ Dispute / Report` — Opens our unified moderation & fact dispute modal.

### 4. Interactive QR-to-Phone Bridge
Text entry with a TV remote is notoriously clunky. Annotated solves this with an enlarged, dynamic QR card pinned to the bottom-right of the screen:
* **Instant Dispute Flow:** Scanning opens the claim on the viewer’s smartphone to review source citations and submit counter-evidence.
* **Second-Screen Annotation:** Family members can capture and publish notes from their phones, which appear live on the TV stream.

### 5. In-App Moderation & Content Safety
Built directly into the 10-foot remote dock:
* Selecting `🚩 Report Abuse & Hide` immediately purges the item from the TV session, dispatches an abuse ticket to our 24-hour moderation queue, displays a toast, and auto-advances.
* Built-in QR codes allow instant review of DMCA forms, Prohibited Content Guidelines, and our transparent Privacy Policy.

---

## 🛠️ How We Built It

We engineered Annotated with a high-performance native-web hybrid architecture designed to maintain 60 FPS on low-power TV chipsets:

* **Native Shell (Kotlin & AndroidX):** A thin native container running on Fire OS 8 (Android 11 / API 30). Hardware acceleration is enforced at the window layer (`hardwareAccelerated = true`), allowing full-bleed 1080p rendering with zero dropped frames.
* **Hardware Remote Interceptor:** Hardware `KeyEvent` signals (`KEYCODE_DPAD_*`, `KEYCODE_ENTER`, `KEYCODE_BACK`) are translated via a bidirectional `AndroidBridge` into instant web events (`handleTvKey`), bypassing clumsy mouse cursor simulation.
* **Web UI Layer (Zero-Bundle Single-File Engine):** Self-contained, glassmorphic 10-foot UI designed with high-contrast typography legible from 10 feet away.
* **Real-time Backend & Media Transcoding:** Built on Supabase (PostgreSQL with Row-Level Security) and AWS S3 for ultra-low latency media delivery.
* **Multi-Platform Notification Engine:** Powered by Resend, delivering responsive notifications across the web, Chrome extension, and Gmail inbox (featuring an in-email notification bell that matches user unread counts).

---

## 🧗 Challenges We Overcame

1. **Remote Navigation Ergonomics:** Standard browser spatial navigation creates focus traps on TV. We engineered a custom directional key router in Kotlin and JavaScript that guarantees deterministic navigation across dock buttons, cards, and modal dialogs.
2. **Audio Autoplay Policies:** Fire OS WebView enforces user gesture policies on media playback. We engineered an intelligent audio manager that configures `mediaPlaybackRequiresUserGesture = false` with an automatic unmute fallback on the first remote gesture, ensuring the TV never sits silent.
3. **Appstore UGC Moderation on TV:** Video feeds containing community submissions require robust moderation. We consolidated abuse reporting, fact disputes, and DMCA claims into a single 10-foot modal that allows viewers to instantly hide objectionable clips on the spot with one click of the remote.

---

## 🏆 Accomplishments We're Proud Of

* **Running Live on Physical Hardware:** Fully tested and sideloaded on an actual **Amazon Fire TV Stick 4K Max 2nd Gen**, delivering silky-smooth 60 FPS transitions.
* **Under 3-Second Cold Start:** Cold start to active HD video playback takes under 2.5 seconds on Fire OS hardware.
* **True Communal Experience:** Proved that TV does not have to be a lonely, passive broadcast — turning the living room into a participatory, shared discovery hub.
* **100% Open Source:** Built cleanly under the MIT License with comprehensive build and run instructions.

---

## 🔮 What's Next for Annotated on Fire TV

* **Alexa Voice Remote Integration:** Saying *"Alexa, ask Annotated to verify this claim"* or *"Alexa, show context"* to trigger the annotation HUD via voice.
* **Family Channel Subscriptions:** Allowing family members to follow each other's channels and sync watch queues across multiple Fire TV sticks in different rooms.
* **Live Communal Watch Parties:** Real-time WebRTC synchronization so friends in different households can watch the same curated stream together with live remote reactions.
