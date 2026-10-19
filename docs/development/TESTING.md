# AI-PEER Testing & Quality Assurance Guide

This guide provides testing protocols, diagnostic checklists, and validation procedures for AI-PEER across mobile native environments, computer vision pipelines, on-device SLM inference, and backend cloud services.

---

## 1. Environment & Preflight Diagnostics

Before executing manual or automated tests, verify that your development environment is healthy:

### iOS Preflight Check
```bash
cd front-end/AI-PEER
npm run ios:doctor
```
Checks performed:
- Node.js version aligns with `.nvmrc` (v24).
- Xcode active path (`xcode-select -p`) points to `/Applications/Xcode.app`.
- CocoaPods version >= 1.16.0.
- `react-native-worklets-core` is present in `node_modules`.
- Podfile C++20 post-install flags are active.

### Android Preflight Check
```bash
cd front-end/AI-PEER
npm run android:doctor
```
Checks performed:
- JDK 17 presence and `JAVA_HOME` configuration.
- Android SDK command-line tools and platform packages installed.
- Gradle daemon memory and build cache sanity.

---

## 2. Mobile App Testing Protocols

> **Important:** Testing camera features (MediaPipe pose estimation) and on-device machine learning (llama.rn) requires a **physical mobile device**. Simulators do not provide camera video frames or GPU/Metal acceleration.

### 2.1 Computer Vision & Pose Tracking Tests
Test file target: `app/(tabs)/exercise-session.tsx`

1. **Camera Stream & Overlay Rendering:**
   - Launch an exercise session (e.g., *Ankle Movements* or *Knee Bends*).
   - Ensure the live video preview renders smoothly at ~30 FPS without frame drops.
   - Confirm that the skeleton overlay aligns with shoulders, hips, knees, and ankles.
2. **Gesture-Confirm Start Flow:**
   - Stand 6–8 feet back from the camera.
   - Raise your open palm / wave hand overhead.
   - Confirm the app transitions from `waiting_for_gesture` to `countdown` (5-4-3-2-1).
   - Confirm countdown audio ticks sound clearly.
3. **Rep Counting State Machine:**
   - Complete 3 repetitions conforming to correct form.
   - Verify the rep counter increments on completion of the eccentric/return phase.
   - Intentionally perform incomplete repetitions (insufficient range of motion) and verify reps are not counted.
4. **Form Violation Feedback:**
   - Intentionally lean excessively or hunch posture.
   - Verify the corresponding violation badge appears on-screen and clears once posture is corrected.

### 2.2 Clinical Assessment Validation

| Assessment Screen | Target Protocol | Verification Criteria |
|---|---|---|
| **`chair-rise-test.tsx`** | 30-Second Sit-to-Stand | Sit on a sturdy armless chair. Verify initial seated state detected. Count each full stand-to-sit cycle. Verify test auto-stops at exactly 30.0 seconds. |
| **`tug-test.tsx`** | Timed Up and Go (TUG) | Start seated, rise, walk 3 meters, turn, and sit down. Verify timing begins upon standing and stops upon reseating. |
| **`balance-test.tsx`** | One-Leg Stand | Lift one foot off the ground. Verify balance timer tracks elapsed single-leg stance duration up to clinical limit. |

### 2.3 On-Device LLM Chat Testing
Test file target: `app/(tabs)/ai-chat.tsx`

1. **First-Run Model Download:**
   - On a clean install, navigate to the AI Chat tab.
   - Verify the download modal appears, showing download progress for `Qwen3.5-2B-aipeer-Q4_K_M.gguf` (~1.2 GB).
   - Confirm successful checksum verification and model file persistence in app data.
2. **Local Inference:**
   - Send health questions (e.g., "What exercises help with knee stiffness?").
   - Verify streaming tokens render in the chat window.
   - Confirm inference completes within ~2–4 seconds per response on modern devices (iPhone 13+, Pixel 7+).
3. **Session Retention & 24h Purge:**
   - Send messages, close the app, and reopen to verify conversation persistence.
   - Confirm that conversations older than 24 hours are automatically purged.

---

## 3. Backend API Testing (`API`)

### 3.1 Local Server Testing
Run the backend locally:
```bash
cd API
npm install
node server.js
```

Validate core endpoints using `curl` or Postman:

```bash
# 1. Health check
curl -X GET http://localhost:3000/health
# Expected: {"status":"OK","timestamp":"..."}

# 2. Public auth endpoints
curl -X POST http://localhost:3000/auth/send-code \
  -H "Content-Type: application/json" \
  -d '{"phoneNumber":"+14075551234","password":"TestPassword123","mode":"login"}'

# 3. Protected endpoint validation (should reject without Bearer token)
curl -X GET http://localhost:3000/activities/user-activities
# Expected: 401 Unauthorized / {"error":"No token provided"}
```

### 3.2 GCS Signed URL Verification
- Call `POST /video/get-video-url` with a valid test Firebase token and `videoId`.
- Copy the returned `videoUrl` and paste into a browser.
- Verify the video streams cleanly and that the URL parameters contain a 1-hour expiration timestamp (`X-Goog-Expires=3600`).

---

## 4. Cloud Functions Testing (`functions`)

Test file target: `functions/index.js` (REDCap sync)

1. **Local Function Emulation:**
   ```bash
   cd functions
   npm install
   npm run serve
   ```
2. **REDCap Mapping Verification:**
   - Verify `functions/config/fieldMappings.js` correctly maps Firestore activity documents to REDCap instrument fields.
   - Run unit test mocks to confirm null handling when assessment scores are omitted.

---

## 5. End-to-End Pull Request QA Checklist

Before merging code to `main`, confirm:
- [ ] `npm run typecheck` passes with zero TypeScript errors.
- [ ] `npm run lint` passes across frontend and backend.
- [ ] `npm run ios:doctor` and `npm run android:doctor` report all PASS.
- [ ] Physical device smoke test passes for exercise session tracking.
- [ ] Translations for `en`, `es`, and `ht` are verified for all new text strings.
