# GemaraSpark 🔥📖

> **AI-powered Talmud study platform — making ancient wisdom accessible through modern technology**

[![Live Demo](https://img.shields.io/badge/Live%20Demo-eldar1.web.app-orange?style=for-the-badge&logo=firebase)](https://eldar1.web.app)
[![Firebase Hosting](https://img.shields.io/badge/Hosted%20on-Firebase-FFCA28?style=for-the-badge&logo=firebase)](https://firebase.google.com)
[![React 18](https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react)](https://react.dev)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)

---

## 🌐 Live Application

**→ [https://eldar1.web.app](https://eldar1.web.app)**

---

## Table of Contents

- [Overview](#overview)
- [Current Features](#current-features)
- [Architecture](#architecture)
- [AI Roadmap — Google AI Integration](#ai-roadmap--google-ai-integration)
- [Getting Started](#getting-started)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

GemaraSpark is a modern, interactive web application designed to make Talmud study (*Gemara*) engaging, accessible, and deeply educational for a new generation of learners. The platform blends traditional Talmudic scholarship with contemporary UX patterns — visual storytelling, gamification, and real-time interactivity — to transform a traditionally text-heavy discipline into an immersive digital experience.

The application is built entirely in Hebrew (RTL layout), targeting Hebrew-speaking users across all skill levels, from complete beginners to advanced scholars.

---

## Current Features

### 1. 📚 Visual Sugyot Explorer (`/sugyot`)
An interactive, step-by-step visual walkthrough of core Talmudic legal debates (*sugyot*).

| Feature | Details |
|---|---|
| **Step Navigation** | Each sugya is broken into discrete logical steps with forward/back navigation |
| **Character Dialogue** | Talmudic sages speak in dramatized, first-person dialogue |
| **Plain-language Explanation** | Every step includes a modern Hebrew explanation of the legal concept |
| **Modern Analogy ("Slang")** | Each step is mapped to a relatable modern-day scenario |
| **Current Sugyot** | *Shnayim Ochzin BaTalit* (BM 2a), *Elu Metziot* (BM 21b) |

**UX Pattern:** Progressive disclosure — learners advance at their own pace, never overwhelmed.

---

### 2. ⚔️ Battle of the Sages (`/battle`)
A gamified debate simulator where users choose a Talmudic sage and watch a dramatized legal argument unfold.

| Feature | Details |
|---|---|
| **Characters** | Abaye (אביי) and Rava (רבא) — the most famous Talmudic debate pair |
| **Character Stats** | Logic, Defense, Passion, Speed — displayed as visual progress bars |
| **Special Moves** | Each character has a unique legal *moveset* derived from their actual Halachic position |
| **Battle Log** | Real-time animated debate transcript styled as Talmudic discourse |
| **Audio Feedback** | Custom Web Audio API synthesized sounds (no external dependencies) |

**Audio Engine:** Fully custom-built using the Web Audio API — oscillators, gain nodes, and frequency ramping — no third-party audio libraries.

---

### 3. 📖 Aramaic Slang Dictionary (`/dictionary`)
A searchable, flip-card dictionary of core Aramaic Talmudic terms.

| Feature | Details |
|---|---|
| **Cards** | 7 core Aramaic terms and phrases |
| **Flip Animation** | CSS 3D `perspective` / `rotateY` card flip on click |
| **Search** | Real-time filtering across Aramaic term, meaning, and modern explanation |
| **Categories** | Terms tagged: `מושגים` (Concepts), `מילות קישור` (Connectors), `ביקורת` (Critique) |

---

### 4. 🏆 Champions Quiz (`/quiz`)
An adaptive quiz module testing Talmudic knowledge with instant feedback.

| Feature | Details |
|---|---|
| **Questions** | 3 curated questions covering core legal concepts and terminology |
| **Answer Feedback** | Correct/incorrect highlighting with full explanation on submission |
| **Scoring** | Point accumulation with streak tracking |
| **Completion Screen** | Score summary with replay option |

---

### 5. 🎨 Design System

| Attribute | Implementation |
|---|---|
| **Framework** | React 18 (UMD/CDN, no build step) with Babel standalone transpilation |
| **Styling** | Tailwind CSS (CDN) + custom CSS for animations and effects |
| **Typography** | Google Fonts — Rubik + Assistant (Hebrew-optimized) |
| **Icons** | Font Awesome 6.4 |
| **Color Palette** | Deep navy (`#0b0f19`) base with amber (`#f59e0b`) primary accent |
| **Animations** | Glow effects, fade-in transitions, 3D card flips, toast notifications |
| **Layout** | Full RTL (`dir="rtl"`) Hebrew layout |
| **Audio** | Custom Web Audio API engine (zero dependencies) |

---

## Architecture

```
gemaraspark/
├── public/
│   └── index.html          # Single-file React application (JSX via Babel standalone)
├── firebase.json           # Firebase Hosting configuration (SPA rewrite rules)
└── .firebase/              # Firebase CLI cache
```

### Data Model (Current — Static)

```javascript
SUGYOT[]          // Talmudic tractate discussions with visual steps
DICTIONARY[]      // Aramaic term definitions
CHARACTERS[]      // Sage profiles with stats and special moves
QUIZ[]            // Question bank with options and explanations
```

All data is currently **hardcoded in the client** — no backend, no database. This is a deliberate MVP decision to achieve zero-cost hosting and maximum simplicity.

### Deployment Stack

```
Browser → Firebase CDN → public/index.html
                      → React 18 (unpkg CDN)
                      → Tailwind CSS (CDN)
                      → Google Fonts (CDN)
```

**Total infrastructure cost: $0/month** (Firebase Spark free tier)

---

## AI Roadmap — Google AI Integration

The following roadmap details a phased plan to integrate **Google AI** services to transform GemaraSpark from a static learning tool into an intelligent, personalized Talmud study platform.

### Vision

> Leverage Google's AI stack to create a 1-on-1 AI study partner (*chavruta*) that adapts to each learner's level, answers questions in real time, generates personalized content, and provides the deepest Talmudic learning experience achievable with 2026 AI technology.

---

### Phase 1 — AI Chavruta (Study Partner) 🤝
**Target:** Q3 2026 | **API:** Gemini 2.5 Pro

The core differentiator: a conversational AI that acts as a traditional Jewish learning partner (*chavruta*), engaging users in Socratic dialogue about the text.

**Features:**
- **`/chat` tab** — Real-time conversational interface styled as a Beit Midrash dialogue
- **Context-aware Q&A** — User selects a sugya; Gemini receives the full Aramaic text + Rashi as system context
- **Socratic method** — AI does not just answer; it asks counter-questions to deepen understanding
- **Multi-level responses** — User sets their level (beginner / intermediate / advanced); AI adjusts vocabulary and depth
- **Source citation** — Every AI claim cites tractate, page, and amora

**Implementation:**
```javascript
// Firebase AI Logic SDK (client-side Gemini integration)
import { initializeApp } from 'firebase/app';
import { getAI, getGenerativeModel } from 'firebase/ai';

const ai = getAI(app);
const model = getGenerativeModel(ai, { model: 'gemini-2.5-pro' });

const systemInstruction = `You are a Talmudic study partner (chavruta).
Current sugya: ${selectedSugya.title} — ${selectedSugya.tractate}
Text: [full Aramaic text + Rashi]
Respond in Hebrew. Use Socratic method. Always cite sources.`;
```

**Security:** Firebase App Check + Firestore security rules to prevent API key exposure.

---

### Phase 2 — Adaptive Learning Engine 🧠
**Target:** Q3 2026 | **APIs:** Gemini 2.5 Flash + Firebase Firestore

Transform the quiz from static to dynamically generated, personalized content.

**Features:**
- **Dynamic question generation** — Gemini generates novel quiz questions from any sugya on demand
- **Mistake analysis** — AI analyzes incorrect answers and explains the exact misconception
- **Spaced repetition** — Firestore stores per-user performance; AI schedules review based on forgetting curves
- **Difficulty calibration** — Questions auto-adjust based on rolling 10-question performance window
- **Learning path** — AI recommends the next sugya based on knowledge gaps

**Data Schema (Firestore):**
```javascript
users/{uid}/
  profile: { level, preferredStyle, totalSessions }
  progress/{sugyaId}: { completed, score, mistakes[], lastSeen }
  quizHistory/{sessionId}: { questions[], answers[], timestamp }
```

---

### Phase 3 — Multimodal Text Understanding 📜
**Target:** Q4 2026 | **API:** Gemini 2.5 Pro Vision

Enable users to photograph physical Talmud pages and receive instant AI-powered explanation.

**Features:**
- **Photo upload** — User photographs a page from a physical Gemara
- **OCR + parsing** — Gemini Vision identifies the tractate, page, and isolates the text from commentaries (Rashi, Tosafot)
- **Layered explanation** — AI provides explanations at three layers: simple reading, legal analysis, philosophical implications
- **Commentator voices** — AI simulates responses in the style of Rashi, Maimonides (Rambam), or Nachmanides (Ramban)
- **Audio narration** — Google Cloud Text-to-Speech reads the AI explanation in natural Hebrew

**UI Flow:**
```
[Upload Photo] → [Gemini Vision: identify page] → [Display detected text]
             → [User selects explanation depth] → [Gemini: generate layered explanation]
             → [Optional: TTS narration] → [Save to personal notebook]
```

---

### Phase 4 — Personalized Content Generation ✍️
**Target:** Q1 2027 | **API:** Gemini 2.5 Pro + Imagen 3

Generate original, personalized learning content tailored to each user.

**Features:**
- **Custom sugyot walkthroughs** — AI generates new visual step-by-step breakdowns for any tractate
- **Personal analogy engine** — User enters their profession/interests; AI generates analogies from their world (e.g., a software engineer gets coding analogies for legal concepts)
- **Story mode** — Gemini generates an immersive narrative short story dramatizing the legal scenario
- **Illustrated cards** — Imagen 3 generates custom illustrations for each Aramaic dictionary term
- **Weekly digest** — Personalized "Daf Yomi"-style summary email generated by Gemini, sent via Firebase Extensions

---

### Phase 5 — Community & Social Learning 🌍
**Target:** Q2 2027 | **APIs:** Gemini + Firebase Realtime Database + Google Meet API

Build a social learning layer on top of the AI foundation.

**Features:**
- **AI-facilitated group chavruta** — Two users study together; AI moderates the discussion and intervenes with hints
- **Debate arena 2.0** — Replace static battle simulation with live AI-generated arguments responsive to user input
- **Community Q&A** — User questions answered by AI, then reviewed/upvoted by human experts
- **AI-generated shiurim (lessons)** — Weekly 5-minute AI video lesson on a featured sugya (Google Cloud Video AI)
- **Leaderboard & Achievements** — Firebase Realtime Database–backed XP system with AI-personalized achievement messages

---

### Google AI Technology Stack (Full Vision)

| Layer | Google Service | Purpose |
|---|---|---|
| **Conversational AI** | Gemini 2.5 Pro | Chavruta dialogue, Socratic Q&A |
| **Fast inference** | Gemini 2.5 Flash | Quiz generation, real-time hints |
| **Vision/OCR** | Gemini 2.5 Pro Vision | Physical page recognition |
| **Image generation** | Imagen 3 | Dictionary card illustrations |
| **Text-to-Speech** | Google Cloud TTS | Hebrew audio narration |
| **Auth** | Firebase Authentication | User accounts (Google Sign-In) |
| **Database** | Cloud Firestore | User progress, quiz history |
| **Hosting** | Firebase Hosting | CDN deployment |
| **Security** | Firebase App Check | Prevent API abuse |
| **AI SDK** | Firebase AI Logic | Client-side Gemini integration |
| **Analytics** | Google Analytics 4 | Learning behavior insights |

---

### Architectural Evolution

**Current (v1.0 — MVP):**
```
Browser → Static HTML/JS → Firebase Hosting CDN
```

**Target (v3.0 — Full AI Platform):**
```
Browser → React SPA → Firebase Auth → Firestore (user data)
                   → Firebase AI Logic → Gemini 2.5 Pro (chavruta)
                   → Cloud Functions → Gemini Flash (quiz gen)
                   → Cloud Storage → Imagen 3 (illustrations)
                   → Firebase Hosting CDN
```

---

## Getting Started

### Prerequisites
- A modern web browser
- (For development) Node.js 18+

### Run Locally

```bash
# Clone the repository
git clone https://github.com/eldarbs/Eldar1.git
cd Eldar1

# No build step required — open directly or use a local server
npx serve public

# Or with Python
python3 -m http.server 8080 --directory public
```

Then open `http://localhost:8080`

### Deploy to Firebase

```bash
npm install -g firebase-tools
firebase login
firebase use eldar1
firebase deploy --only hosting
```

---

## Contributing

Contributions are welcome! Areas where help is especially valuable:

- 📚 **Content** — Additional sugyot, dictionary terms, and quiz questions
- 🤖 **AI Integration** — Firebase AI Logic / Gemini API implementation (see Phase 1 roadmap)
- 🎨 **Design** — Mobile optimization, accessibility (a11y) improvements
- 🌍 **Localization** — English and French UI translations

Please open an issue before submitting a large PR to align on approach.

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

<div align="center">

Built with ❤️ to make the wisdom of the Talmud accessible to every generation

**[eldar1.web.app](https://eldar1.web.app)**

</div>
