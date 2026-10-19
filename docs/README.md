# AI-PEER Documentation Hub

Welcome to the AI-PEER documentation repository. AI-PEER is a HIPAA-compliant mobile application developed by UCF Senior Design in collaboration with the UCF College of Medicine (CoM). It provides on-device fall-risk assessment and Otago exercise interventions for older adults.

This directory serves as the centralized documentation library for all technical architecture, operational runbooks, clinical protocols, and compliance standards.

---

## Documentation Quick Reference

| Category | Document | Audience | Description |
|---|---|---|---|
| **Architecture** | [architecture/ARCHITECTURE.md](architecture/ARCHITECTURE.md) | All Developers, Architects | System overview, end-to-end data flows, Firestore schema, auth architecture, and core design principles. |
| **Architecture** | [architecture/FILE_MAP.md](architecture/FILE_MAP.md) | All Developers | Comprehensive inventory of every source file across backend, mobile frontend, native modules, and ML pipelines. |
| **Development** | [development/SETUP.md](development/SETUP.md) | New Contributors | Step-by-step developer onboarding for macOS, Node 24, Xcode 26, Android Studio, and environment variables. |
| **Development** | [development/TROUBLESHOOTING.md](development/TROUBLESHOOTING.md) | Developers, DevOps | Diagnostic runbook for C++20 Clang module caches, VisionCamera build errors, Android Gradle daemons, and Cloud Run. |
| **Development** | [development/TESTING.md](development/TESTING.md) | QA, Developers | Testing protocols for mobile native builds, doctor preflight scripts, clinical assessments (TUG, Chair Rise), and backend APIs. |
| **Development** | [development/CONTRIBUTING.md](development/CONTRIBUTING.md) | Contributors | Git branching strategies, PR submission checklist, coding standards, and Scrum sprint tracking conventions. |
| **Pipelines** | [pipelines/VISION_PIPELINE.md](pipelines/VISION_PIPELINE.md) | Mobile & CV Engineers | Deep dive on MediaPipe Pose + Hand Landmarkers, gesture-confirm start flow, skeleton overlays, and rep counter state machines. |
| **Pipelines** | [pipelines/LLM_PIPELINE.md](pipelines/LLM_PIPELINE.md) | ML & Mobile Engineers | Architecture of on-device Qwen3.5-2B model via llama.rn, GCS model downloads, prompt engineering, and 24h conversation TTL. |
| **Pipelines** | [pipelines/AUDIO_PIPELINE.md](pipelines/AUDIO_PIPELINE.md) | Mobile & Audio Engineers | Design specification for real-time spoken form feedback, audio queue throttling, STT senior input, and multilingual TTS voices. |
| **Compliance** | [compliance/HIPAA_COMPLIANCE.md](compliance/HIPAA_COMPLIANCE.md) | DevOps, Compliance, Legal | Technical and administrative safeguards, on-device compute privacy guarantees, audit logging, and PHI masking. |
| **Compliance** | [compliance/Allowed BAA.txt](compliance/Allowed%20BAA.txt) | DevOps, CoM IT | Inventory of Google Cloud Platform services covered under the institutional Business Associate Agreement. |
| **Deployment** | [deployment/PRODUCTION_MIGRATION.md](deployment/PRODUCTION_MIGRATION.md) | DevOps, UCF CoM IT | Complete hand-off plan for deploying AI-PEER into a production GCP project owned by UCF College of Medicine IT. |
| **Clinical** | [clinical/GLOSSARY.md](clinical/GLOSSARY.md) | Sponsors, Non-Developers | Plain-English definitions of clinical terms (Otago, BTrackS, FES-I, FRA) and technical frameworks. |
| **Tutorials** | [tutorial/README.md](tutorial/README.md) | Clinic Staff, Translators | Source Markdown and automated PDF build pipelines for patient print handouts in English, Spanish, and Haitian Creole. |
| **Media** | [media/](media/) | Maintainers | Video demonstrations, feature walkthroughs, and repository media assets. |

---

## Directory Organization

```
docs/
├── architecture/         # System design, data flow diagrams, and exhaustive file inventories
│   ├── ARCHITECTURE.md
│   └── FILE_MAP.md
├── pipelines/            # Specialized engineering deep-dives for on-device AI & CV
│   ├── VISION_PIPELINE.md
│   ├── LLM_PIPELINE.md
│   └── AUDIO_PIPELINE.md
├── development/          # Contributor onboarding, testing guidelines, and troubleshooting
│   ├── SETUP.md
│   ├── TROUBLESHOOTING.md
│   ├── TESTING.md
│   └── CONTRIBUTING.md
├── compliance/           # HIPAA policies, privacy guarantees, and BAA contracts
│   ├── HIPAA_COMPLIANCE.md
│   └── Allowed BAA.txt
├── deployment/           # Production infrastructure cutover and institutional hand-off
│   └── PRODUCTION_MIGRATION.md
├── clinical/             # Clinical protocols, exercise taxonomy, and sponsor glossaries
│   └── GLOSSARY.md
├── tutorial/             # Multilingual patient education handouts and pandoc PDF build scripts
│   ├── README.md
│   ├── build.sh
│   └── tutorial_{en,es,ht}.md
└── media/                # Demo screencasts, feature walkthroughs, and video clips
    └── LLM First Integration - Made with Clipchamp.mp4
```

---

## Navigation by Persona

### 👨‍💻 New Developer / Contributor
1. Begin with the [development/SETUP.md](development/SETUP.md) guide to configure Node 24, Xcode, CocoaPods, and environment variables.
2. Read [architecture/FILE_MAP.md](architecture/FILE_MAP.md) to understand where each screen, component, and backend route lives.
3. Review [development/CONTRIBUTING.md](development/CONTRIBUTING.md) for branch naming, coding style, and PR workflow.
4. Keep [development/TROUBLESHOOTING.md](development/TROUBLESHOOTING.md) handy when running builds.

### 📱 Mobile & Vision Engineer
1. Study [pipelines/VISION_PIPELINE.md](pipelines/VISION_PIPELINE.md) for the frame processor architecture, angle math, and rep counting logic.
2. Review [pipelines/AUDIO_PIPELINE.md](pipelines/AUDIO_PIPELINE.md) for live voice cues and debounced speech alerts during workouts.
3. Consult the frontend guide in [front-end/AI-PEER/README.md](../front-end/AI-PEER/README.md).

### 🤖 Machine Learning / SLM Engineer
1. Read [pipelines/LLM_PIPELINE.md](pipelines/LLM_PIPELINE.md) for the on-device inference runtime with `llama.rn`.
2. Inspect `Training/slm/TRAINING_PLAN.md` for fine-tuning datasets, QLoRA hyperparameters, and GGUF quantization.

### 🏥 Clinical Partner / CoM IT Administrator
1. Consult [deployment/PRODUCTION_MIGRATION.md](deployment/PRODUCTION_MIGRATION.md) for GCP project provisioning, service accounts, and cutover steps.
2. Read [compliance/HIPAA_COMPLIANCE.md](compliance/HIPAA_COMPLIANCE.md) and [compliance/Allowed BAA.txt](compliance/Allowed%20BAA.txt) for privacy safeguards and covered services.
3. Reference [clinical/GLOSSARY.md](clinical/GLOSSARY.md) for clinical assessment mappings (FES-I, TUG, 30s Chair Rise, FRA Matrix).
4. See [tutorial/README.md](tutorial/README.md) for patient educational handouts.

---

## Documentation Maintenance Rules

When making architectural or structural changes to the codebase:
- Keep [architecture/FILE_MAP.md](architecture/FILE_MAP.md) synchronized when adding or renaming source files.
- Update [development/SETUP.md](development/SETUP.md) if prerequisites or toolchain versions change.
- Add newly identified build gotchas or runtime fixes to [development/TROUBLESHOOTING.md](development/TROUBLESHOOTING.md).
- Ensure all clinical updates remain consistent with [clinical/GLOSSARY.md](clinical/GLOSSARY.md) and translation files.
