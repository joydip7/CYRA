# CYRA 📩
## A Voice-First AI Career & Autonomous Inbox Agent

CYRA is an AI-powered, career-centric email agent designed to transform traditional inbox management into an intelligent decision engine. It helps students and professionals prioritize important emails, extract actionable career insights, automate repetitive inbox tasks, and optimize cold outreach — all through a voice-first interface.

---

## 📚 Documentation & Architecture

This project follows a modular, agent-based architecture with clear separation between reasoning and execution.

The complete blueprint is available below:

| Document | Description |
|----------|------------|
| 📘 [Requirements Specification](.docs/requirements.md) | User stories, MVP scope, agent capabilities, safety constraints |
| 🏗️ [System Design](.docs/design.md) | Architecture diagram, Gmail API integration, LangGraph flow |

---

## 🚀 Key Features (MVP)

### 📥 Smart Email Prioritization Engine
- Automatically classifies emails into:
  - Urgent
  - Job/Internship
  - Promotional
  - Meetings
  - General
- Provides intelligent summary:
  > “You have 12 unread emails. 2 urgent. 3 job-related.”

### 🎓 Career Intelligence Mode
- Detects internship and job opportunities
- Extracts:
  - Role
  - Company
  - Deadline
  - Location
  - Stipend/Salary
  - Required skills
- Presents concise structured summaries

### 🤖 Autonomous Decision Mode
- Auto-archive promotional emails
- Star job-related emails
- Highlight urgent messages
- Suggest context-aware replies

### ✉️ Cold Email Optimizer
- Tone analysis
- Personalization score
- Spam probability
- Response likelihood prediction

### 🗣️ Voice-First Interface
- Speech-to-text input
- Text-to-speech output
- Fallback to text input

### 📊 Productivity Dashboard
- Emails auto-filtered
- Career opportunities detected
- Promotions removed
- Time saved metrics

---

## 🛡️ Safety & Privacy

- OAuth-based Gmail integration
- No permanent storage of email bodies
- Explicit confirmation before destructive actions
- Strict reasoning vs execution separation

---

## 🏗️ System Architecture Overview

User (Voice/Text)
↓
Intent Detection (LLM)
↓
Email Classification Engine
↓
LangGraph Agent Logic
↓
Execution Layer
↓
Gmail API
↓
Text + Voice Output

---

## 🛠️ Tech Stack

**Frontend**
- React.js (Vite)
- Tailwind CSS
- Web Speech API

**Backend**
- Python (FastAPI)

**AI/ML**
- Google Gemini API / OpenAI
- LangGraph

**Database**
- PostgreSQL / SQLite

**Integrations**
- Gmail API
- Google OAuth 2.0

---

## ⚠️ Usage Disclaimer

CYRA is an AI-powered productivity assistant.  
It does not replace professional communication judgment.  
All automated actions can be confirmed or overridden by the user.

---

## 🎯 Vision

Transforming inbox management from a passive notification stream into an intelligent, autonomous career decision system.
