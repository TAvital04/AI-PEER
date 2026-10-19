# Real-Time Audio & Speech Pipeline

This document details the architecture and operational design of the real-time voice feedback and hands-free speech interface in AI-PEER.

Developed as part of Sprint 2 (`AUDIO-EPIC-2`), this subsystem directly addresses fall risks for older adults (age 60+) by eliminating the need to look closely at screens or tap mobile buttons while performing standing balance and strength exercises.

---

## 1. Clinical Rationale & Accessibility Need

When an older adult performs Otago exercises (e.g., Heel-to-Toe Stand, Knee Bends, One-Leg Stand) or clinical balance assessments (30-second Chair Rise, Timed Up and Go) at a distance of 6 to 10 feet from the phone:

1. **Visual Hazard:** Craning the neck to read small badge overlays or text prompts on a phone display shifts the user's center of gravity and creates an immediate fall risk.
2. **Physical Friction:** Stepping toward the phone to tap touch targets interrupts the movement rhythm and compromises test timing.
3. **Cognitive Fatigue:** Seniors with tremor or arthritic stiffness find typing on small virtual keyboards stressful during cognitive reframing and questionnaire entry.

The voice pipeline delivers a **hands-free, eyes-up experience** combining real-time spoken coaching, intelligent alert debouncing, and speech-to-text input.

---

## 2. Architecture Overview

```
                                  +---------------------------------------+
                                  | Vision Pipeline (MediaPipe 30 FPS)    |
                                  |   FormAnalyzer -> FormFeedback        |
                                  +---------------------------------------+
                                                      |
                                                      | Form Violations & Rep Events
                                                      v
  +--------------------+          +---------------------------------------+
  | User Preferences   | -------> | Audio Queue Manager                   |
  | - Sound enabled    |          | - Priority classification             |
  | - Locale (en/es/ht)|          | - Violation debouncer (3s cooldown)   |
  | - Speech rate      |          | - Alert throttling & queue dedupe     |
  +--------------------+          +---------------------------------------+
                                                      |
                                                      | Ordered Utterances
                                                      v
                                  +---------------------------------------+
                                  | Text-to-Speech (TTS) Engine           |
                                  |   src/tts.ts (expo-speech)            |
                                  |   en-US | es-ES | fr-FR (for ht)      |
                                  +---------------------------------------+
```

---

## 3. Subsystem Components

### 3.1 Spoken Live Form Correction Engine (`AUDIO-1`)
- **Hook Point:** Connected directly into `VisionContext.tsx` and `FormAnalyzer.ts`.
- **Form Violations:** When the pose analyzer detects form breakdown (such as excessive trunk lean, inadequate knee flexion, or unlevel shoulders), it translates the violation type into a localized spoken phrase.
  - *Example (English):* "Keep your chest up." / "Stand tall." / "Hold on to a sturdy chair."
  - *Example (Spanish):* "Mantén el pecho erguido." / "Párate derecho."
  - *Example (Haitian Creole):* "Kenbe pwatrin ou dwat." / "Rete dwat."
- **Rep Milestones:** Spoken milestones announce cadence without stepping on safety warnings (e.g., "Rep 5", "Halfway there", "Workout complete!").

### 3.2 Audio Queue Manager & Debouncer (`AUDIO-3`)
Without throttling, a 30 FPS camera loop could emit hundreds of alerts per minute, overwhelming older adults. The `AudioQueueManager` applies strict rate controls:

- **Priority Hierarchy:**
  1. **Emergency / Safety Alerts (Priority 1):** Unsafe posture or loss of balance cues (e.g., "Grab your chair"). Interrupts lower-priority speech immediately.
  2. **Form Corrections (Priority 2):** Minor postural adjustments (e.g., "Straighten knees"). Enforces a 3-second cooldown per distinct violation type.
  3. **Rep Milestones (Priority 3):** Rep counter increment announcements. Queued only when no safety alert is active.
  4. **Motivational / Cadence Cues (Priority 4):** General encouragement ("Great job, keep going"). Dropped if the queue is non-empty.
- **Debounce Window:** Consecutive violation triggers must sustain across at least 300 ms of video frames before qualifying for speech queue insertion.

### 3.3 Multilingual TTS Locale Routing (`AUDIO-4`)
Text-to-speech output is centralized through `src/tts.ts` using `expo-speech`:

| In-App Language Code | Target TTS Voice Locale | Notes |
|:---:|:---:|---|
| `en` | `en-US` | Native US English voice |
| `es` | `es-ES` / `es-US` | Native Spanish voice |
| `ht` | `fr-FR` | French voice fallback (iOS does not bundle a native Haitian Creole synthesizer) |

All utterance strings are loaded from `front-end/AI-PEER/src/locales/{en,es,ht}/translation.json` under the `audio.*` key namespace to guarantee synchronized multilingual support.

### 3.4 Speech-to-Text (STT) for Senior Accessibility (`AUDIO-2`)
- **Use Cases:**
  - Hands-free navigation during assessment setups ("I am ready", "Start test", "Repeat instructions").
  - Voice-driven conversational inputs for the on-device AI cognitive reframing chat (`app/(tabs)/ai-chat.tsx`).
  - Answering FES-I fall-efficacy questionnaire items hands-free.
- **Senior-Optimized Acoustic Tuning:**
  - Extended silence detection timeout (minimum 2.5 seconds pause required before auto-submit) to accommodate slower speech cadence.
  - Visual microphone state indicators with high-contrast active pulsing animations.

---

## 4. Integration & Testing Guidelines

When testing audio features:
1. **Device Speaker Verification:** Run on physical iOS/Android test devices; audio output behaviors differ on simulators.
2. **Audio Interruption Testing:** Validate that phone ringers, alarms, or system notifications do not crash the TTS worklet or freeze the camera preview.
3. **Mute / Preference Respect:** Confirm that toggling "Sound Alerts" OFF in `app/(tabs)/settings.tsx` suppresses all exercise speech cues instantly.
