# Agentic Systems Engineering Portfolio

I design and operate AI agent systems in production. The case studies below document real systems I've built and run — multi-channel conversational agents, scheduled human-in-the-loop alert systems, and the production debugging that comes with both.

This portfolio focuses on what I've actually shipped, including the bugs I caused and how I diagnosed them.

## How I got here

I'm Dr. Collins, founder of Collins Wellness — a remote holistic wellness clinic. The clinic's operational backbone is AI agent orchestration: lead qualification, client onboarding, daily protocol coaching, scheduled check-ins, and outage detection all run on n8n with Anthropic Claude as the LLM layer. What started as "I need to stop answering the same intake questions" has become a multi-system platform I designed, built, debug, and operate.

I'm now applying the same skill set to engineering roles building agentic systems in other domains.

## Case Studies

### [01 — Multi-Mode Conversational Agent](./01-unified-conversational-agent.md)

A single n8n workflow that handles three different conversation types — new prospects, qualified leads, and active subscribers — across Telegram and WhatsApp, with state-driven routing between modes. Documents the architecture, the design decisions, and four real production bugs including the n8n `promptType` silent-reset issue, timezone-safe streak math, and the WhatsApp 24-hour window constraint.

### [02 — Multi-Channel Check-In Alert System](./02-checkin-alert-system.md)

A human-in-the-loop alert system that surfaces clients due for outreach across four notification channels. Anniversary-based trigger logic, phase-review escalation for clinical clients, and noon-anchored timestamps for DST safety. Includes honest documentation of a production gap I surfaced *by writing this case study*.

### [03 — WhatsApp API Outage Postmortem](./03-whatsapp-api-outage.md)

A two-layer production bug — expired Meta token and Business Account ID mismatch — diagnosed by absence-detection, fixed via System User token migration, and followed up with a daily health-check workflow. The closing principle: when fixing a bug changes the symptom instead of removing it, you uncovered the next bug. Keep going.

## Stack

- **Orchestration:** n8n (self-hosted)
- **LLM:** Anthropic Claude
- **State / data:** Google Sheets (prototype scale), planned Postgres migration at production scale
- **Channels:** Telegram Bot API, WhatsApp Business API (Meta Graph), Gmail, Google Calendar
- **CRM:** vCita

## In progress

A code-first reimplementation of the Coach mode (case study #1) using LangGraph for orchestration and MCP (Model Context Protocol) for tool exposure — bridging the n8n production work above with the frameworks most modern agentic systems teams use. Link will be added here when shipped.

## Contact

- Email: collinsengineering04@gmail.com
- LinkedIn: *[your LinkedIn URL]*
- Website: *[website if you want to link it]*
