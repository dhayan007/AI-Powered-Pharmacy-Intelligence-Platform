# Pharmacy Intelligence Platform

**A full-stack pharmacy operations platform with a predictive analytics engine and a natural-language AI data assistant — turning day-to-day pharmacy data into decisions a business user can act on without writing a query.**

`Python` `FastAPI` `SQLAlchemy` `PostgreSQL` `React` `Vite` `LLM / RAG` `AI-to-SQL`

> **A note on this repo:** this documents the architecture, design decisions, and functionality of a system I built end-to-end. The full source is kept private to protect implementation details built for a real operational use case — screenshots, demo videos, and this write-up show the working system in detail, and I'm glad to walk through the code directly in an interview.

---

## At a glance

- **Full-stack system**, not a notebook: React/Vite frontend → FastAPI backend → SQLAlchemy ORM → PostgreSQL
- **Predictive layer**: demand forecasting, reorder prediction, and expiry-risk scoring built on historical operational data
- **AI assistant**: users ask questions in plain English and get answers computed live from the database — no SQL required — via a controlled, validated AI-to-SQL pipeline
- **End-to-end ownership**: schema design, backend services, analytics logic, predictive models, and the AI layer were all built by me

---

## Problem Statement

Pharmacies generate significant amounts of operational data through daily activities such as sales, inventory updates, supplier transactions, and purchase orders.

However, this data is often distributed across operational systems and is not effectively used for decision-making.

 Common challenges include:

- Difficulty identifying products that are running out of stock
- Expiry-related inventory losses
- Manual inventory monitoring
- Limited visibility into sales and inventory performance
- Difficulty analysing large volumes of operational data
- Inefficient purchase-order planning
- Limited access to data-driven business insights

This project addresses these challenges by combining operational management with analytics and intelligent decision-support capabilities.

---

## Key features

**Inventory intelligence**
Real-time stock tracking, low-stock alerts, near-expiry detection, and product-level inventory analysis — surfacing problems before they hit daily operations.

**Sales & business analytics**
Dashboards covering sales performance, inventory value, top-selling products, stock movement, cashflow, and supplier-level analytics.

**Supplier & purchase order management**
End-to-end procurement workflow — supplier management, purchase order creation and tracking, and supplier payments — connected directly to inventory intelligence rather than run as a separate system.

**Predictive intelligence**
- *Reorder prediction* — flags products likely to need replenishment based on historical sales and inventory patterns
- *Expiry-risk detection* — identifies inventory at risk of expiring unsold, ahead of time
- *Purchase recommendations* — surfaces suggested purchase-order decisions from demand and inventory signals

**AI-powered data assistant**
A conversational layer that lets users ask operational questions in natural language and get answers computed live from the database:

> *"Which products generated the highest sales this month?"*
> *"What products are currently running low on stock?"*
> *"Which inventory items are approaching expiry?"*

No SQL knowledge required on the user's end, and the assistant tracks conversation context — a follow-up like *"what about yesterday?"* is understood relative to the prior question, not treated as a standalone query. See the pipeline below for how that's handled.




---

## System architecture

```
                     ┌──────────────────────┐
                     │      React UI         │
                     │    Vite Frontend      │
                     └──────────┬────────────┘
                                │
                                ▼
                     ┌──────────────────────┐
                     │     FastAPI API       │
                     │  Business Logic       │
                     │  Analytics Services    │
                     │  AI Services           │
                     └──────────┬────────────┘
                                │
                                ▼
                     ┌──────────────────────┐
                     │      SQLAlchemy        │
                     │    ORM / Data Layer    │
                     └──────────┬────────────┘
                                │
                                ▼
                     ┌──────────────────────┐
                     │      PostgreSQL         │
                     │  Operational Database   │
                     └──────────────────────┘
```

## AI assistant pipeline

The assistant doesn't call the LLM straight through to the database — every question passes through a controlled pipeline that classifies intent, resolves conversational context, and validates any SQL before it touches the database:

```
User Question
      │
      ▼
Conversation Memory
      │
      ▼
Intent Classification
      │
   ┌──┼──┐
   ▼  ▼  ▼
  SQL RAG CHAT
   │
   ▼
Follow-up Detection
      │
      ▼
Question Rewriting
      │
      ▼
Prompt Builder
      │
      ▼
     LLM
      │
      ▼
SQL Generation
      │
      ▼
 SQL Cleaner
      │
      ▼
SQL Validator
      │
      ▼
 PostgreSQL
      │
      ▼
Response Builder
      │
      ▼
     LLM
      │
      ▼
Natural Language Answer
      │
      ▼
Conversation Memory
```

**Why this matters:** the assistant doesn't treat every message as a fresh question. **Intent classification** first decides whether a message needs a database query (SQL), retrieved context (RAG), or is just conversational (CHAT). For SQL-bound queries, **follow-up detection** checks whether the new message depends on prior turns — if it does, **question rewriting** resolves it into a fully self-contained query before it ever reaches the LLM, and the result is written back into conversation memory for the next turn.

**Example:**

> **User:** "What were today's sales?"
> **Assistant:** *(runs the query, returns today's total)*
>
> **User:** "What about yesterday?"
> **Assistant:** *(detects this is a follow-up, rewrites it internally to "What were yesterday's sales?", and answers correctly — no need for the user to restate the full question)*

The validation step exists specifically because the assistant executes generated SQL against a live database — it's a deliberate guardrail, not an afterthought.

---

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | React, Vite |
| Backend | Python, FastAPI |
| Data layer | SQLAlchemy ORM, PostgreSQL |
| Analytics | SQL, BI dashboarding, demand/reorder/expiry-risk models |
| AI | LLMs, RAG, prompt construction, AI-to-SQL, conversational memory, SQL validation |

---

## Results & status

This is a self-built, self-tested project — there hasn't been a formal evaluation against production data, so the numbers below are about scope and functional coverage rather than accuracy metrics I can't honestly claim yet.

- **Functional end-to-end**, from raw operational data through to dashboards, predictive scoring, and the AI assistant — every layer in the architecture diagram above is implemented and working, not stubbed.
- **AI assistant** handles multi-turn conversations with working follow-up detection (see the "today's sales" → "what about yesterday?" example above), tested manually across a range of natural-language questions during development.
- **Predictive models** (reorder prediction, expiry-risk scoring) run against historical operational data I generated/loaded for development and testing, rather than live production data from a real pharmacy.

*I'm planning a more rigorous evaluation pass — backtesting the predictive models against held-out data and running the assistant against a structured set of test questions to get real accuracy figures. Happy to discuss the current testing approach and what a proper evaluation would look like.*

---

## Repository contents

```
Ai-Powered-Pharmacy-Intelligence-Platform/
├── README.md
├── LICENSE.md
├── .gitignore
│
├── docs/
│   ├── system-architecture.png
│   ├── ai-assistant-pipeline.png
│   ├── data-pipeline.png
│   └── technical-design.md
│
├── screenshots/
│   ├── sales-overview-dashboard.png
│   ├── inventory-alerts.png
│   ├── billing.png
│   ├── inventory.png
│   ├── cashflow-dashboard.png
│   └── chatbot.png
│
└── demo/
    ├── dashboard-demo.mp4
    └── chatbot-demo.mp4
```

## About this project

Built independently to explore the intersection of full-stack engineering, data modelling, analytics, and applied AI — treating pharmacy operations and intelligence as one connected system rather than a management tool with a reporting layer bolted on.

**Mohammed Dhayan Ahmed** — [LinkedIn](https://linkedin.com/in/mohammed-dhayan-ahmed-a509b7250) · dhayan343@gmail.com
