# WhatsApp API Outage: Token Migration, Business Account Mismatch, and the Health Check It Earned

A production debugging story — the day every WhatsApp send in my system failed at once, what I found, and what I changed afterward.

## TL;DR

All WhatsApp send operations across my workflows started failing simultaneously. First hypothesis (Meta service issue) was wrong. Real cause was a two-layer bug: my access token had expired (standard Meta tokens are short-lived), *and* the Business Account ID I'd configured in n8n didn't match the WABA the new token was authorized against. Migrated to a permanent System User token, corrected the Business Account ID, and built a daily health-check workflow so the same class of failure can't go undetected again.

## What happened

I noticed I hadn't received a check-in message in a couple of days. That was the first signal — and it was a soft one, easy to miss. I reached out to a few clients to see if they'd been getting their messages either. None of them had. That ruled out anything user-specific and pointed straight at the system.

Opened n8n, checked recent executions on the WhatsApp workflows, and saw every WhatsApp send node was failing — the conversational bot's WhatsApp leg, the supplement reminders, the check-in alert, all of them. Telegram and Gmail nodes kept working fine, so it was clearly Meta-specific, not a network issue or an n8n-wide problem.

The detection story matters here: nothing alerted me. I noticed because *my own* messages stopped, and I confirmed it because *clients* hadn't gotten theirs. That gap — the time between the actual failure and me figuring it out — is the gap the health check at the end of this writeup was built to close.

## First hypothesis (wrong)

My first guess was a Meta service-side issue. I checked Meta's status page. Nothing. Then I assumed I'd hit a rate limit — maybe I'd accidentally fanned out too many sends in a short window. The error responses didn't quite match rate-limit responses though, so that didn't hold up either.

## Root cause #1: Token expiration

The Meta error pointed to my access token being expired. Standard Meta access tokens are short-lived by design — they expire on a fixed cadence and roll over silently. The token I'd originally configured wasn't a System User token; it was a regular user-scoped token, and it had quietly rolled.

The right primitive for production automation is a **System User token**. Differences that matter:

- Don't expire on a user-level cadence (effectively permanent until you rotate them)
- Are scoped to specific assets (in my case, the WhatsApp Business Account)
- Survive the user's session lifecycle

The migration: I created a System User in Meta Business Settings, assigned it the right permissions on the WhatsApp Business Account, generated a token scoped to that asset, and swapped the credential in n8n. Named it `Collins_Wellness_Bot` so I'd recognize it later.

## Root cause #2: Business Account ID mismatch

While testing the new token, the error changed. The token was now valid, but the API was rejecting requests with a different error — a Business Account access issue.

The mismatch: the Business Account ID I'd originally entered into n8n wasn't actually the WABA the System User token was authorized against. This is a remarkably easy mistake to make because Meta's UI surfaces multiple IDs that all look similar at a glance — Meta Business ID, App ID, WABA ID, Phone Number ID, System User ID. They're all 15-16 digit numeric strings, and only some are documented clearly.

The fix: pulled the correct WABA ID from WhatsApp Manager, updated the credential in n8n, sends started working immediately.

The whole outage took longer to diagnose than to fix, which is the usual ratio for these.

## What I added afterward

A **daily health-check workflow** for the WhatsApp API. It runs a no-op send (or a self-test ping) once a day and alerts me on Telegram if it fails. The original outage went undetected for *days* because nothing was actively monitoring the API health — I caught it the way users catch outages, by noticing the absence of expected behavior.

Since the corrections, the health check has been quiet. The underlying fixes (System User token, correct WABA ID) have held — no new incidents to report. But quiet is exactly what I want from a health check; it's an insurance policy, not a feature. The day it does fire is the day it pays for itself, and I'd rather have it sit silent for a year than not have it the next time something rolls.

## What I'd do differently

**Use System User tokens from day one.** I'd been building fast and used the regular token because it was right there in the Meta UI when I first wired up WhatsApp. The migration cost me hours of debugging that wouldn't have existed if I'd started with the right primitive. Now I default to System User tokens for any production integration, even when I'm just prototyping.

**Verify which IDs are which before connecting.** Meta's UI is genuinely confusing on the various ID types. Now when I'm setting up a new Meta integration I keep a labeled note — "this 16-digit string is the WABA ID, this one is the Phone Number ID" — so I don't conflate them later. Sounds basic, but it's saved me twice since.

**Build health checks for every external API integration, not just the one that bit me.** The WhatsApp health check is good but I should have similar pings on every third-party API I depend on — Telegram Bot, Google Sheets, Calendar, Gmail, vCita. If any of them fail silently, I should know within a day, not whenever a downstream symptom shows up. This is on the roadmap.

## Lessons

This was a textbook example of **two bugs hiding behind one symptom**. The token expiration would have been simple to fix on its own. The Business Account ID mismatch would have been simple to fix on its own. Together, the second one only surfaced *after* I'd fixed the first, which made the whole thing feel longer and more confusing than the actual time spent fixing either issue.

The general principle: when you fix a bug and the symptom changes instead of disappearing, you didn't fix the bug — you uncovered the next one in line. Keep going.

Second lesson, less elegant but more important: **detection-by-absence is not detection.** The fact that I noticed the outage by missing my own message means I'm running blind on the things I haven't been tracking actively. The health check workflow exists because I never want to discover an outage that way again.

## Stack / artifacts

- **Meta Business Manager** — System User creation, asset assignment, permanent token generation
- **n8n credentials updated** for: conversational bot workflow, supplement reminder workflow, check-in alert workflow
- **New workflow:** Daily WhatsApp Health Check (cron-triggered ping, Telegram alert on failure)

---

*Companion debugging story to the [Multi-Mode Conversational Agent](./01-unified-conversational-agent.md) and [Multi-Channel Check-In Alert](./02-checkin-alert-system.md) — both of those systems have WhatsApp integrations that depend on the work described here.*
