# AI-PEER HIPAA Compliance & Security Architecture

This document outlines the technical, administrative, and physical security safeguards implemented in AI-PEER to ensure compliance with the Health Insurance Portability and Accountability Act (HIPAA), Health Information Technology for Economic and Clinical Health (HITECH) Act, and UCF College of Medicine (CoM) institutional security standards.

---

## 1. Compliance Philosophy: Zero-Egress On-Device Processing

The fundamental architectural principle of AI-PEER is that **sensitive patient biometrics and free-form patient conversations never leave the mobile device**.

```
                +-------------------------------------------------------+
                |                    Mobile Device                      |
                |                                                       |
                |  Camera Stream ---> MediaPipe Pose + Hand Landmarker  |
                |                     (In-memory frames; NO GCS upload) |
                |                                                       |
                |  Patient Chat  ---> On-Device Qwen3.5-2B (llama.rn)   |
                |                     (Local inference; NO cloud LLM)   |
                |                                                       |
                |  Local Cache   ---> 24-Hour TTL Auto-Purge            |
                +-------------------------------------------------------+
                                           |
                                  TLS 1.3 Encrypted
                                  (Metadata / Reps only)
                                           v
                +-------------------------------------------------------+
                |             Google Cloud Platform (BAA Covered)       |
                |  - Firestore: Activity counts, rep scores, FES-I      |
                |  - Cloud Run: Stateless Express API                   |
                |  - GCS: Read-only instructional exercise videos       |
                +-------------------------------------------------------+
```

### Key Technical Privacy Guarantees

1. **No Video Recording or Egress:**
   - The device camera feed is processed in real time by the native frame processor plugin (`PoseLandmarkerPlugin.swift` on iOS, `PoseLandmarkerFrameProcessorPlugin.kt` on Android).
   - Video frames are held temporarily in volatile memory buffers for landmark extraction and immediately dropped.
   - Raw video is never saved to the local file system, never cached in persistent app storage, and never transmitted to Google Cloud Storage or any external API.

2. **On-Device Natural Language Processing:**
   - The AI conversational assistant utilizes an on-device Small Language Model (`Qwen3.5-2B-aipeer-Q4_K_M.gguf`, ~1.2 GB) loaded via `llama.rn`.
   - Patient prompts, health queries, and model responses run 100% locally on the device's CPU/GPU/Neural Engine.
   - No patient prompts or conversational inputs are sent to OpenAI, Anthropic, Google Vertex AI, or any third-party cloud LLM.

3. **24-Hour Ephemeral Chat Retention:**
   - Conversation histories in `app/chat-history.tsx` and AsyncStorage carry an automatic 24-hour Time-To-Live (TTL).
   - Once the TTL expires, conversations are permanently purged from local storage, minimizing the exposure window on lost or shared devices.

---

## 2. Technical Safeguards (45 CFR § 164.312)

### Access Control & Authentication
- **Multi-Factor Authentication (MFA):** All user logins and registrations require mandatory two-factor authentication via SMS OTP (Google Identity Platform).
- **Two-Token Persistence Model:**
  - Standard Firebase ID tokens are short-lived (1 hour).
  - A cryptographic UUID refresh token is securely stored in client `AsyncStorage` and validated against Firestore (`refreshTokens` collection) with a strict 30-day lifespan.
  - Revocation of a refresh token in Firestore immediately terminates mobile access across cold starts.
- **Rate Limiting & Abuse Prevention:**
  - SMS verification endpoints enforce a 60-second cooldown between consecutive codes and a hard maximum of 5 attempts per hour per phone number.

### Transmission Security & Data at Rest
- **Encryption in Transit:** All client-to-backend and backend-to-database communications occur over HTTPS / TLS 1.3. Plain HTTP connections are rejected.
- **Encryption at Rest:**
  - Google Cloud Firestore Native mode automatically encrypts all database collections at rest using Google-managed or customer-managed encryption keys (AES-256).
  - Google Cloud Storage buckets (`aipeer_videos`, `qwenfinetune`) enforce AES-256 server-side encryption at rest.
- **Time-Limited Signed URLs:**
  - Instructional exercise videos stored in Google Cloud Storage are never made publicly readable.
  - The API issues time-limited signed URLs generated via IAM `signBlob` with a strict 1-hour expiration. Private service account keys do not need to reside on the production server.

### Audit Controls & PHI De-identification
- **Phone Number Masking:** Application logs produced by Cloud Run and Cloud Functions automatically mask patient phone numbers. For example, `+14075551234` is transformed to `XXXX1234` prior to log emission.
- **Minimal Metadata Logging:** API request logging records HTTP methods, status codes, response latencies, and sanitized user IDs (`uid`), completely omitting request bodies containing questionnaire answers or health metrics.

---

## 3. Business Associate Agreement (BAA) Coverage

AI-PEER operates exclusively on Google Cloud Platform services covered by UCF's institutional Business Associate Agreement.

### Covered Services in Active Use

| GCP Service | Role in AI-PEER | Covered Under Google BAA |
|---|---|:---:|
| **Cloud Run** | Hosts the Express 5.1 backend API container | ✅ Yes |
| **Cloud Firestore** | Stores patient profiles, activity histories, and FRA matrix results | ✅ Yes |
| **Cloud Storage** | Stores signed instructional exercise videos and model GGUF weights | ✅ Yes |
| **Cloud Functions (Gen 2)** | Scheduled nightly sync between REDCap and Firestore | ✅ Yes |
| **Identity Platform** | SMS 2FA phone number verification and token generation | ✅ Yes |
| **Secret Manager** | Houses private credentials and API keys in secure vaults | ✅ Yes |
| **Cloud Scheduler** | Triggers automated cron jobs (REDCap sync) | ✅ Yes |
| **Cloud Logging** | Audit trail storage with automatic log retention policies | ✅ Yes |
| **Cloud Monitoring** | System uptime and health tracking | ✅ Yes |
| **Cloud IAM** | Least-privilege service account permission management | ✅ Yes |

The complete list of GCP services certified under the UCF BAA is maintained in [Allowed BAA.txt](Allowed%20BAA.txt).

---

## 4. Administrative & Operational Safeguards

### Institutional Hand-off & Environment Isolation
- **Separate Production Infrastructure:** The production environment is deployed into an isolated GCP project owned exclusively by UCF College of Medicine IT (`research-ai-peer-prod`).
- **Separation of Student and Clinical Access:**
  - Student engineering team accounts are restricted to the development project (`research-ai-peer-dev`).
  - Student developers possess zero administrative, IAM, or data viewer roles within the production GCP project.
  - Ownership is assigned to institutional Google Groups (`aipeer-prod-owners@ucf.edu`, `aipeer-prod-admins@ucf.edu`).

### Clinical Data Integration (REDCap)
- Clinical trial fall-risk assessment scores (BTrackS, FES-I) and intervention compliance metrics are synchronized bi-directionally between Firestore and UCF's HIPAA-compliant REDCap server.
- The synchronization function (`functions/index.js`) executes over encrypted channels on a nightly schedule via Cloud Scheduler, minimizing daytime network footprint and human interaction.

---

## 5. Security Incident Response & Contact

In the event of a suspected vulnerability, unauthorized access, or compliance question, notify:

- **UCF College of Medicine Information Security Office:** `comsecurity@ucf.edu`
- **UCF CoM IT Operations:** `aipeer-prod-admins@ucf.edu`
- **Reference Runbook:** [PRODUCTION_MIGRATION.md](../deployment/PRODUCTION_MIGRATION.md) (Section 3 & Section 13)
