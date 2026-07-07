# AI-Powered Facebook Messenger Bot — n8n Automation System

**Production-grade, dual-AI Facebook Messenger automation built on n8n, Google Gemini, and Groq — handling real customer conversations end-to-end without human intervention.**

[![Automation](https://img.shields.io/badge/Automation-n8n-EA4B71?style=flat-square)](https://n8n.io)
[![AI Model](https://img.shields.io/badge/Primary%20AI-Gemini%202.5%20Flash-4285F4?style=flat-square)](https://ai.google.dev)
[![Fallback AI](https://img.shields.io/badge/Fallback%20AI-Groq%20Llama%203.3%2070B-F55036?style=flat-square)](https://groq.com)
[![Platform](https://img.shields.io/badge/Platform-Meta%20Messenger%20API-0866FF?style=flat-square)](https://developers.facebook.com/docs/messenger-platform)
[![Status](https://img.shields.io/badge/Status-Production-brightgreen?style=flat-square)]()
[![Maintained by](https://img.shields.io/badge/Maintained%20by-AutomateIQ%20Labs-black?style=flat-square)](https://www.facebook.com/automateiq.labs/)

> 📌 **This is a documentation / case-study repository.** It showcases the architecture, workflow design, and decision logic of a live AI Messenger bot through screenshots and technical write-up. Source code is not published — see [Why No Source Code](#why-no-source-code-is-public) below.

---

## Table of Contents

- [Overview](#overview)
- [The Problem It Solves](#the-problem-it-solves)
- [System Architecture](#system-architecture)
- [How It Works](#how-it-works)
- [Dual-AI Brain Architecture](#dual-ai-brain-architecture)
- [Tech Stack](#tech-stack)
- [Key Features](#key-features)
- [White-Label Deployment Model](#white-label-deployment-model)
- [Capacity & Results](#capacity--results)
- [FAQ](#faq)
- [Why No Source Code Is Public](#why-no-source-code-is-public)
- [Connect](#connect)

---

## Overview

This repository documents an **AI Messenger bot for Facebook Pages** — a production automation system that answers customer messages instantly, in the customer's own language, using a live knowledge base instead of scripted replies. It runs on a **14-node n8n workflow** with two separate AI agents working in sequence: one to reply, one to silently decide whether a human needs to step in.

It was designed as a **white-label AI automation template** — one workflow, redeployable across multiple Facebook Business Pages by swapping only the knowledge base and business identity.

**Core keywords:** AI Messenger bot, n8n workflow automation, Facebook Graph API chatbot, Gemini AI agent, Groq Llama fallback, conversational AI for business, customer service automation, lead detection AI, WhatsApp/Messenger automation Bangladesh.

## The Problem It Solves

Small and medium businesses lose leads every day because nobody is online to reply outside business hours. A customer who messages at 3 AM and gets silence usually doesn't come back. This system closes that gap:

- Instant, accurate replies — 24/7, no human on standby
- Replies pulled from real business data (Google Docs knowledge base), not hardcoded FAQ scripts
- Automatic language detection (Bangla / English) with no manual configuration
- Serious inquiries are flagged and escalated to a human — casual messages are not

## System Architecture

```mermaid
flowchart TD
    A[Facebook Messenger User] --> B[Meta Webhook]
    B --> C[n8n Webhook Trigger]
    C --> D[Input Data Extraction]
    D --> E{Data Firewall: Text or Media?}
    E -->|Media/Sticker| F[Send Warning Reply]
    E -->|Text| G{Casual Message Filter}
    G -->|Casual: hi/hello/ok| H[Fixed Reply Template]
    G -->|Genuine Query| I[AI Brain 1: Response Engine]
    I -->|Reads| J[(Knowledge Base — Google Docs)]
    I --> K[Reply Sent via Graph API]
    K --> L[AI Brain 2: Conversation Analyst]
    L --> M{Decision}
    M -->|NOTIFY_ADMIN| N[Admin Alert via Messenger]
    M -->|MONITOR / NO_ACTION| O[Session Ends]
```

📸 *Full n8n canvas screenshot:*

`assets/screenshots/workflow-overview.png`

*(Replace this file with your own exported workflow screenshot — see [assets/screenshots](assets/screenshots))*

## How It Works

1. **Webhook receives** the incoming Messenger event from Meta's Graph API
2. **Data extraction** pulls `user_message`, `sender_id`, `recipient_id`, and `page_id`
3. **Data firewall** filters out non-text input (images, stickers, voice notes) with a polite fallback message
4. **Casual-message filter** short-circuits greetings (`hi`, `hello`, `হাই`, `ok`) into a fixed branded reply — saving AI quota for real queries
5. **AI Brain 1** (Gemini 2.5 Flash, with Groq Llama 3.3 70B as automatic fallback) reads the knowledge base and generates a contextual, language-matched reply, appending an invisible routing tag
6. **Reply is sent** to the user instantly via the Graph API
7. **AI Brain 2** re-reads the full session and independently decides: `NOTIFY_ADMIN`, `MONITOR`, or `NO_ACTION`
8. **Admin is notified on Messenger** only when a decision genuinely warrants it — not for every message

## Dual-AI Brain Architecture

The core design decision behind this system: **one AI model should not both talk to the customer and judge the conversation.** Splitting those responsibilities into two agents removes bias and keeps notifications meaningful.

| | AI Brain 1 — Response Engine | AI Brain 2 — Conversation Analyst |
|---|---|---|
| **Job** | Generate the customer-facing reply | Decide if a human needs to intervene |
| **Input** | User message + Knowledge Base | Full session history + latest reply |
| **Output** | Natural-language reply + hidden tag | Structured JSON decision |
| **Runs** | Once per message | Once per message, after the reply is sent |
| **Visible to user** | Yes | No — fully silent |

This separation means the customer never sees a robotic "let me connect you to a human" fallback — the analyst layer handles that decision quietly in the background.

## Tech Stack

| Layer | Technology |
|---|---|
| Automation Engine | [n8n](https://n8n.io) (self-hosted) |
| Messaging Platform | Meta Facebook Graph API v25.0 |
| Primary AI Model | Google Gemini 2.5 Flash |
| Fallback AI Model | Groq — Llama 3.3 70B |
| Knowledge Base | Google Docs (structured, AI-readable) |
| Hosting | AWS EC2 (Ubuntu 22.04) |
| Reverse Proxy / SSL | Nginx + Let's Encrypt (Certbot) |

## Key Features

- 🌐 Automatic bilingual language detection (Bangla / English) — no manual switch
- 🧠 Live knowledge-base retrieval instead of static/hardcoded replies
- 🏷️ Hidden intent tagging system (`[LEAD_DETECTED]`, `[ADMIN_NEEDED]`, `[OFF_TOPIC]`) invisible to the end user
- 🔁 Automatic AI model fallback — the bot never goes silent if one provider rate-limits
- 🧵 Per-user session memory (last 10 messages) for context-aware replies
- 🔔 Smart, de-duplicated admin notifications — one alert per issue, not per message
- 🧱 Media/sticker firewall with a graceful fallback response
- 💸 $0/month infrastructure cost using free-tier services throughout

## White-Label Deployment Model

This workflow was built to be **redeployed for new clients in under 15 minutes** by changing only two things:

| Changes per client | Stays identical |
|---|---|
| Knowledge Base document | Full 14-node n8n workflow |
| Fixed reply templates & branding | AI Brain 1 & Brain 2 prompt logic |
| Page Access Token | Message filtering & routing rules |

## Capacity & Results

- **Rate limit headroom:** ~2,500 AI requests/day combined (Gemini + Groq), against a real-world load of ~17 requests/day for 500 monthly users
- **Response latency:** sub-5-second reply time from message receipt to delivery
- **Operating cost:** $0/month (fully within free-tier limits across all services)
- **Live status:** deployed and actively serving real customer conversations on a business Facebook Page

## FAQ

**What AI models power this Messenger bot?**
Google Gemini 2.5 Flash is the primary model, with Groq's Llama 3.3 70B running as an automatic fallback so replies never stop if one provider is rate-limited.

**How does the bot decide when to escalate to a human?**
A second AI agent ("AI Brain 2") reads the full conversation after every exchange and outputs a structured decision — `NOTIFY_ADMIN`, `MONITOR`, or `NO_ACTION` — independently of the reply-generating agent.

**What automation platform is this built on?**
[n8n](https://n8n.io), a self-hosted workflow automation engine, using a 14-node pipeline connected directly to the Meta Facebook Graph API.

**Can this system be adapted for other businesses or Facebook Pages?**
Yes — it was designed as a white-label template. Only the knowledge base and branding change; the workflow, AI prompts, and routing logic stay the same.

**Does this bot support languages other than English?**
Yes. It auto-detects the customer's language (currently Bangla and English) per message and replies in kind, without any manual configuration.

**Is this a chatbot builder / SaaS product?**
No — this is a custom-built automation system deployed for a specific business, documented here as a technical case study.

## Why No Source Code Is Public

This repository intentionally contains **no workflow export or source code**. The n8n workflow, system prompts, and knowledge base structure were built as commercial, client-deployable IP for [AutomateIQ Labs](https://www.facebook.com/automateiq.labs/), and are not open-sourced. What you'll find here is the architecture, decision logic, and a screenshot of the live system for portfolio and case-study purposes.

If you're a business owner or agency interested in a similar system for your own Facebook Page, reach out via the contact details below.

## Connect

**Muhammad Antor** — AI Automation Builder, Bangladesh 🇧🇩

- LinkedIn: [linkedin.com/in/muhammad-antor](https://www.linkedin.com/in/muhammad-antor)
- Facebook (AutomateIQ Labs): [facebook.com/automateiq.labs](https://www.facebook.com/automateiq.labs/)
- Instagram: [instagram.com/automateiq.labs](https://www.instagram.com/automateiq.labs/)
- GitHub: [github.com/muhammadantor](https://github.com/muhammadantor)

---

<sub>Documentation repository by AutomateIQ Labs — "Automate Smarter with AI". Screenshots and architecture shared for portfolio purposes; underlying implementation is proprietary client work.</sub>
