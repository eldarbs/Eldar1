# GemaraSpark AI Roadmap 🤖📖

> Technical specification for integrating Google AI services into the GemaraSpark Talmud learning platform.

**Document Status:** Draft v1.0 | Last Updated: July 2026  
**Author:** GemaraSpark Engineering  
**Audience:** Contributors, stakeholders, and AI integration engineers

---

## Executive Summary

GemaraSpark's AI strategy is built on a single principle: **the best teacher is one who knows you**. Google's AI ecosystem — led by Gemini 2.5 Pro and the Firebase AI Logic SDK — enables us to build a platform that adapts in real time to each learner's knowledge level, learning style, and cognitive gaps.

The Talmud is humanity's greatest documented intellectual debate. With modern AI, we can make every user feel like they have a personal Rosh Yeshiva available 24/7 — one who never tires, never judges, and always finds the perfect analogy for your world.

---

## Phase 1: AI Chavruta — Conversational Study Partner

### Overview
A *chavruta* (חברותא) is the traditional Jewish method of paired Talmud study, where two learners challenge each other through dialogue. Phase 1 replaces the static "Battle" feature with a live AI conversation partner powered by Gemini 2.5 Pro.

### User Story
> As a learner studying *Shnayim Ochzin BaTalit*, I want to ask the AI "but what if one of them is lying?" and receive a response that quotes the actual Talmudic discussion about oaths (*shevuot*) and explains why the rabbis designed the solution the way they did.

### Technical Implementation

#### 1.1 Firebase AI Logic SDK Setup

```javascript
// firebase-config.js
import { initializeApp } from 'firebase/app';
import { getAI, getGenerativeModel, Schema } from 'firebase/ai';

const firebaseConfig = {
  apiKey: process.env.VITE_FIREBASE_API_KEY,
  authDomain: 'eldar1.firebaseapp.com',
  projectId: 'eldar1',
  // ...
};

const app = initializeApp(firebaseConfig);
const ai = getAI(app); // Uses Firebase AI Logic (Gemini backend)
export const geminiPro = getGenerativeModel(ai, { model: 'gemini-2.5-pro' });
export const geminiFlash = getGenerativeModel(ai, { model: 'gemini-2.5-flash' });
```

#### 1.2 System Prompt Architecture

The quality of the chavruta experience depends entirely on the system prompt. We use a layered injection strategy:

```javascript
function buildChavrutaSystemPrompt(sugya, userLevel) {
  return `
## Role
You are a Talmudic study partner (chavruta) with expertise equivalent to a musmach 
of a top Israeli yeshiva. You are warm, intellectually rigorous, and deeply enjoy debate.

## Active Sugya
Title: ${sugya.title}
Tractate: ${sugya.tractate}
Aramaic Text: [INJECTED_AT_RUNTIME]
Rashi's Commentary: [INJECTED_AT_RUNTIME]
Tosafot Notes: [INJECTED_AT_RUNTIME]

## User Level: ${userLevel}
${userLevel === 'beginner' ? 
  'Use only simple Hebrew. Avoid untranslated Aramaic. Use lots of modern analogies.' :
  userLevel === 'intermediate' ?
  'Use Aramaic terms with inline explanations. Reference other tractates sparingly.' :
  'Engage as a peer scholar. Use Aramaic freely. Reference rishonim and acharonim.'}

## Pedagogical Rules
1. NEVER simply answer a question — always respond with a counter-question first
2. If the user is wrong, guide them to discover the error themselves (Socratic method)
3. Every factual claim MUST cite: tractate name, page (daf), and amora who said it
4. When the user reaches a correct insight, celebrate it enthusiastically
5. Offer "deeper dive" opportunities: "Do you want to see how the Rambam ruled on this?"
6. Respond in Hebrew. Use Aramaic only when quoting the text directly.

## Prohibited Behaviors
- Do not answer questions outside the scope of Talmud/Halacha
- Do not invent source citations — say "I'm not certain of the exact source" if needed
- Do not express opinions on contemporary political or religious controversies
`.trim();
}
```

#### 1.3 Conversation State Management

```javascript
// Using Gemini's multi-turn chat API
const chat = geminiPro.startChat({
  systemInstruction: buildChavrutaSystemPrompt(selectedSugya, userLevel),
  history: previousMessages, // Restored from Firestore for session continuity
  generationConfig: {
    maxOutputTokens: 800,
    temperature: 0.7,      // Balanced: creative but not hallucinating
    topP: 0.9,
  },
  safetySettings: [
    { category: 'HARM_CATEGORY_HATE_SPEECH', threshold: 'BLOCK_LOW_AND_ABOVE' },
  ],
});

const response = await chat.sendMessage(userMessage);
```

#### 1.4 UI Component Design

```
┌─────────────────────────────────────────────┐
│  🔥 Chavruta Mode — שניים אוחזין בטלית      │
│  Level: [Beginner] [Intermediate] [Advanced] │
├─────────────────────────────────────────────┤
│                                             │
│  🤖 AI: "נניח שמצאת טלית ברחוב ועוד         │
│     מישהו מרים אותה בו-זמנית — מה הדין?"   │
│                                             │
│  👤 You: "כנראה חולקים?"                    │
│                                             │
│  🤖 AI: "מעניין! ולמה דווקא חצי חצי?        │
│     מה בדיוק ה-CLAIM של כל צד?"             │
│                                             │
├─────────────────────────────────────────────┤
│  [Type your response...]          [Send ⚡] │
│  💡 Hint  📚 Sources  🔊 Read aloud         │
└─────────────────────────────────────────────┘
```

---

## Phase 2: Adaptive Learning Engine

### 2.1 Dynamic Quiz Generation

Replace static quiz questions with Gemini-generated questions for any sugya:

```javascript
// Structured output using Gemini's JSON schema
const QuizQuestionSchema = Schema.object({
  question: Schema.string(),
  options: Schema.array(Schema.object({
    text: Schema.string(),
    isCorrect: Schema.boolean(),
    explanation: Schema.string(),
  })),
  difficulty: Schema.enumString(['easy', 'medium', 'hard']),
  concept: Schema.string(), // The core concept being tested
  sourceReference: Schema.string(), // e.g., "BM 2a"
});

async function generateQuizQuestion(sugya, difficulty, avoidConcepts) {
  const model = getGenerativeModel(ai, {
    model: 'gemini-2.5-flash',
    generationConfig: {
      responseMimeType: 'application/json',
      responseSchema: Schema.array(QuizQuestionSchema),
    },
  });

  const prompt = `Generate 3 multiple-choice quiz questions about the Talmudic sugya: 
    "${sugya.title}" (${sugya.tractate}).
    Difficulty: ${difficulty}.
    Avoid these already-tested concepts: ${avoidConcepts.join(', ')}.
    Each question must test a distinct legal or conceptual insight from the sugya.
    Wrong options should represent common misconceptions, not obviously incorrect answers.`;

  const result = await model.generateContent(prompt);
  return JSON.parse(result.response.text());
}
```

### 2.2 Spaced Repetition Algorithm

```javascript
// SM-2 algorithm adapted for Talmudic concepts
function calculateNextReview(performance, previousInterval, easeFactor) {
  // performance: 0-5 (0=complete blackout, 5=perfect recall)
  const newEaseFactor = Math.max(1.3, 
    easeFactor + (0.1 - (5 - performance) * (0.08 + (5 - performance) * 0.02))
  );
  
  const newInterval = performance < 3 
    ? 1  // Re-study today
    : previousInterval === 1 ? 6
    : previousInterval === 6 ? 15
    : Math.round(previousInterval * newEaseFactor);
    
  return { nextInterval: newInterval, newEaseFactor };
}

// Firestore schema for tracking
// users/{uid}/reviews/{conceptId}: {
//   nextReviewDate: Timestamp,
//   interval: number (days),
//   easeFactor: number,
//   repetitions: number,
// }
```

---

## Phase 3: Multimodal — Physical Page Recognition

### 3.1 Gemini Vision Integration

```javascript
async function analyzeTalmudPage(imageFile) {
  const model = geminiPro; // Vision capability included in gemini-2.5-pro
  
  // Convert file to inline data
  const imageData = await fileToBase64(imageFile);
  
  const result = await model.generateContent([
    {
      inlineData: {
        mimeType: imageFile.type,
        data: imageData,
      },
    },
    `You are an expert in Talmudic manuscripts and printed editions.
    
    Analyze this image of a Talmud page and return a JSON response with:
    1. tractate: The tractate name (מסכת)
    2. daf: Page number and side (e.g., "ב ע\"א")
    3. mainText: The core Gemara text (center column)
    4. rashiText: Rashi commentary (right column, if visible)
    5. tosafotText: Tosafot commentary (left column, if visible)
    6. confidence: Your confidence level (0-1)
    
    If this is not a Talmud page, return { error: "Not a Talmud page" }.`,
  ]);
  
  return JSON.parse(result.response.text());
}
```

### 3.2 Layered Explanation Pipeline

```
Image Upload
    ↓
Gemini Vision: OCR + Structure Parsing
    ↓
Identified: Tractate + Page + Text Layers
    ↓
User Selects Explanation Depth:
  [Simple Reading] → Gemini Flash: plain Hebrew translation
  [Legal Analysis] → Gemini Pro: detailed legal breakdown
  [Philosophical]  → Gemini Pro: broader implications + rishonim
    ↓
Optional: Google Cloud TTS → Hebrew audio narration
    ↓
Save to: Firestore /users/{uid}/notebook/{pageId}
```

---

## Phase 4: Personalized Content Generation

### 4.1 Personal Analogy Engine

```javascript
async function generatePersonalizedAnalogy(concept, userProfile) {
  const prompt = `
  Explain the Talmudic legal concept of "${concept.name}" 
  (${concept.tractate}) to someone whose profession is: ${userProfile.profession}
  and whose hobbies include: ${userProfile.interests.join(', ')}.
  
  Create an analogy that uses scenarios directly from their world.
  The analogy must:
  - Be accurate to the legal nuance (no oversimplification)
  - Use specific, concrete examples from their field
  - Take max 3 sentences
  - End with how the analogy breaks down (its limits)
  
  Respond in Hebrew.`;
  
  const result = await geminiFlash.generateContent(prompt);
  return result.response.text();
}
```

### 4.2 Imagen 3 — Dictionary Card Illustrations

```javascript
// Cloud Function (server-side only — never expose Imagen API client-side)
exports.generateDictionaryIllustration = onCall(async (request) => {
  const { term, meaning } = request.data;
  
  const imageResponse = await imagen.generateImages({
    model: 'imagen-3.0-generate-002',
    prompt: `A minimalist, dark-themed digital illustration representing 
    the Aramaic Talmudic concept "${term}" which means "${meaning}".
    Style: modern flat design, deep navy background (#0b0f19), 
    amber gold (#f59e0b) accents. No text in image. 
    Abstract but conceptually accurate. Museum quality.`,
    numberOfImages: 1,
    aspectRatio: '1:1',
    safetyFilterLevel: 'BLOCK_ONLY_HIGH',
  });
  
  // Store in Firebase Storage
  const imageUrl = await uploadToStorage(imageResponse.images[0], `dictionary/${term}`);
  return { imageUrl };
});
```

---

## Phase 5: Community Features

### 5.1 AI-Moderated Group Chavruta

```javascript
// Multi-user session with AI moderator using Gemini Live API
const session = await geminiPro.startLiveSession({
  systemInstruction: `You are a Rosh Yeshiva moderating a chavruta session 
  between two students. Monitor their discussion. Intervene only when:
  1. They reach a factual error — gently correct with a source citation
  2. They get stuck for >60 seconds — offer a guiding hint
  3. They reach a breakthrough — celebrate and deepen the discussion
  4. The discussion drifts off-topic — redirect warmly`,
  
  // Real-time Firebase Realtime Database for multi-user sync
  onMessage: (msg) => broadcastToSession(sessionId, msg),
});
```

---

## Security Architecture

### Firebase App Check
All Gemini API calls go through Firebase App Check to prevent quota abuse:

```javascript
import { initializeAppCheck, ReCaptchaV3Provider } from 'firebase/app-check';

const appCheck = initializeAppCheck(app, {
  provider: new ReCaptchaV3Provider(process.env.VITE_RECAPTCHA_KEY),
  isTokenAutoRefreshEnabled: true,
});
```

### Rate Limiting Strategy

| User Tier | Gemini Pro calls/day | Gemini Flash calls/day |
|---|---|---|
| Anonymous | 5 | 20 |
| Free (Google Sign-In) | 50 | 200 |
| Premium | Unlimited | Unlimited |

Rate limiting enforced via Firebase Cloud Functions + Firestore counters (not client-side).

### Data Privacy
- Conversation history stored in Firestore with user-owned security rules
- No conversation data shared between users
- Users can delete all their data via Settings → Delete Account
- No PII sent to Gemini API (user ID anonymized before injection)

---

## Migration Plan: CDN → Vite Build System

The current single-file architecture will migrate to a proper Vite + React build as AI features require npm packages:

```bash
# Phase 1 migration
npm create vite@latest gemaraspark-v2 -- --template react
cd gemaraspark-v2
npm install firebase
npm install @google/generative-ai  # Fallback if Firebase AI Logic unavailable

# Build and deploy
npm run build
firebase deploy --only hosting
```

**Migration is backward-compatible** — the `public/index.html` CDN version remains live throughout the migration.

---

## Success Metrics

| Metric | Current Baseline | Phase 1 Target | Phase 3 Target |
|---|---|---|---|
| Avg. session duration | ~3 min (static) | 15 min | 30 min |
| Sugya completion rate | N/A | 60% | 80% |
| Return visit rate (7-day) | Unknown | 25% | 50% |
| Questions asked per session | 0 | 5 | 12 |
| User-reported comprehension | N/A | 4.0/5.0 | 4.5/5.0 |

---

## Open Questions

- [ ] Should the AI chavruta speak in formal or colloquial Hebrew?
- [ ] Do we need a Rabbi/posek review layer before AI legal answers go live?
- [ ] Should we support Sephardic/Ashkenazic Halachic tradition preferences?
- [ ] How do we handle controversial topics (e.g., women's status in Halacha)?
- [ ] Offline mode priority — PWA with cached Gemini responses?

---

*This document is a living specification. Open a GitHub Issue to propose changes.*
