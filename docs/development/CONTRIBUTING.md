# Contributing to AI-PEER

Thank you for contributing to the AI-PEER project! AI-PEER is a joint initiative between UCF Senior Design and the UCF College of Medicine, building an accessible, HIPAA-compliant fall-risk assessment and Otago exercise intervention mobile platform.

To maintain code quality, security compliance, and smooth developer onboarding, please follow these guidelines when submitting changes.

---

## 1. Development Principles

1. **HIPAA & Privacy First:** Never commit code that uploads raw camera video frames, persists unencrypted conversation logs, or bypasses on-device model inference.
2. **Deterministic Builds:** Respect pinned versions (Node 24 in `.nvmrc`, React Native 0.81.5, CocoaPods 1.16+). Avoid introducing host-dependent global dependencies.
3. **Accessibility & Usability:** Our primary users are adults aged 60 and older. All UI elements must maintain high contrast, accommodate font scaling, and prioritize clear audio feedback.
4. **Multilingual Parity:** User-facing strings must be added to all three translation catalogs in `front-end/AI-PEER/src/locales/` (`en`, `es`, `ht`).

---

## 2. Git & Branching Strategy

The repository follows a trunk-based development workflow with short-lived feature branches:

- **`main`** — Production-ready trunk. All merges require an approved Pull Request and passing status checks.
- **Branch Naming Conventions:**
  - `feature/<ticket-id>-<short-description>` (e.g., `feature/CICD-1-doc-reorganization`)
  - `fix/<ticket-id>-<short-description>` (e.g., `fix/AUDIO-3-speech-queue-overflow`)
  - `docs/<short-description>` (e.g., `docs/add-testing-guide`)
  - `refactor/<short-description>` (e.g., `refactor/vision-context-state-machine`)

---

## 3. Commit Message Standards

We use standard Conventional Commits with Scrum/Jira issue keys where applicable:

```
<type>(<scope>): <short description> [TICKET-KEY]

[optional body explaining rationale and approach]
```

### Supported Types
- `feat`: New feature or user story (e.g., `feat(audio): add real-time spoken form feedback (AUDIO-1)`)
- `fix`: Bug fix (e.g., `fix(ios): resolve Clang module cache error in post_install hook`)
- `docs`: Documentation updates or additions (e.g., `docs: consolidate root markdown into /docs (CICD-1)`)
- `refactor`: Code changes that neither fix a bug nor add a feature
- `test`: Adding or updating test suites
- `chore`: Dependency updates, build script improvements, or repository hygiene

---

## 4. Pre-Flight Checks & Quality Assurance

Before pushing a branch or opening a Pull Request, run the local diagnostics:

### Frontend Mobile App (`front-end/AI-PEER`)
```bash
cd front-end/AI-PEER

# 1. Type safety check
npm run typecheck

# 2. ESLint linting
npm run lint

# 3. Environment pre-flight (read-only diagnostic)
npm run ios:doctor       # Verifies Xcode, CLT, CocoaPods, C++20 flags
npm run android:doctor   # Verifies JDK 17, Android SDK, Gradle env
```

### Backend API (`API`)
```bash
cd API

# Verify dependencies and local server boot
npm install
node server.js
# Confirm GET http://localhost:3000/health returns status OK
```

---

## 5. Documentation Maintenance Checklist

Whenever you modify repository layout or features:
- [ ] Update [FILE_MAP.md](../architecture/FILE_MAP.md) if files were added, deleted, or moved.
- [ ] Update [SETUP.md](SETUP.md) if new environment variables or tools are required.
- [ ] Add new diagnostic solutions to [TROUBLESHOOTING.md](TROUBLESHOOTING.md) if you resolved a non-trivial build error.
- [ ] Check that links in [README.md](../../README.md) and [docs/README.md](../README.md) remain valid.

---

## 6. Pull Request Submission Checklist

When opening a Pull Request:
- [ ] **Title:** Include conventional commit prefix and Scrum issue key (e.g., `feat(audio): Spoken live form feedback and exercise coaching (AUDIO-1)`).
- [ ] **Description:** Detail what changed, why, and how you validated it.
- [ ] **Screenshots / Recordings:** For mobile UI changes, attach a screenshot or screen recording.
- [ ] **No Secrets:** Confirm `.env`, service account JSONs, or API keys are not committed.
- [ ] **Clean Diff:** Verify no accidental formatting diffs or untracked binary files were included.
