# Appointment Reminder & Confirmation System

An n8n automation for dental/healthcare clinics that automatically sends appointment reminders to patients, reads their replies using AI, and updates a tracking sheet — without staff manually texting or calling every patient on the schedule.

Built as a hands-on automation project to solve a common clinic problem: **no-shows and missed confirmations caused by manual, easy-to-forget reminder processes.**

---

## The Problem

Clinics typically book appointments into a calendar or sheet, then rely on staff to remember to call or text each patient the day before. This doesn't scale, gets skipped when the front desk is busy, and leaves confirmations untracked — clinics often only find out an appointment isn't happening when the patient simply doesn't show up.

This project automates the entire loop: send the reminder, read the reply, update the record — all without manual intervention.

---

## How It Works

The system is split into two separate workflows, since each is triggered by a fundamentally different type of event.

### Workflow A — Reminder Sender (time-based)

```
Schedule Trigger (runs daily)
        ↓
Google Sheets — Get Rows (reads all appointments)
        ↓
IF Node — is this appointment tomorrow AND not already reminded?
        ↓
Send Reminder Email
        ↓
Google Sheets — Update Row (marks Status as "Reminder Sent")
```

Runs automatically on a schedule, checks the appointments sheet for anything happening the next day, and sends a reminder — but only once per appointment, even if the schedule runs multiple times before the appointment date passes.

**Key logic:** the IF node checks two conditions together (AND, not OR) — the appointment must be tomorrow's date, *and* it must not already have "Reminder Sent" as its status. This prevents the same patient from being reminded repeatedly.

### Workflow B — Reply Handler (event-based)

```
Trigger (simulated incoming reply)
        ↓
Edit Fields — captures which patient + what they replied
        ↓
AI Agent — classifies the reply: Confirmed / Reschedule / Unclear
        ↓
Switch Node — maps classification to a readable status label
        ↓
Google Sheets — Update Row (writes final status back)
```

When a patient replies to their reminder, an AI Agent reads the message and classifies it into one of three categories based on its content — not just keyword matching, but actual language understanding (e.g., "can we do Tuesday instead?" is correctly read as a reschedule request, not just checked for the word "reschedule"). The result is mapped to a clean, human-readable label before being written back to the sheet.

---

## Tech Stack

- **n8n** (workflow automation)
- **Google Sheets** (appointment data store and status tracker)
- **Google Gemini** (AI Agent's language model)
- **Gmail** (reminder delivery)

---

## n8n Nodes Used

- Schedule Trigger
- Google Sheets (Get Rows, Update Row)
- IF
- Gmail (Send Email)
- Manual Trigger
- Edit Fields (Set)
- AI Agent
- Structured Output Parser
- Switch

---

## Key Design Decisions

- **Two separate workflows, not one:** the reminder-sending half runs on a timer, while the reply-handling half is triggered by an external event (a patient's reply). These are fundamentally different trigger types, so splitting them keeps each workflow simple to reason about and debug independently.
- **Duplicate-prevention built in from the start:** the Status column doubles as a safeguard — once an appointment is marked "Reminder Sent," the IF node blocks it from being processed again on subsequent runs.
- **Row matching by `row_number`, not by name:** Google Sheets' Update Row operation matches on a unique row identifier rather than the patient's name, avoiding the (real) risk of two patients sharing a name and the wrong row being updated.
- **AI classification over rigid keyword rules:** the reply-reading AI Agent uses natural language understanding to classify intent, allowing it to correctly interpret varied phrasing rather than requiring patients to reply with exact expected words.

---

## Status

This is a working prototype built as a learning project. Workflow B currently uses a simulated reply trigger (Manual Trigger with manually entered test data) in place of a live SMS/email reply integration — a real deployment would replace this trigger with a webhook connected to an actual messaging service, without needing to change the AI classification or sheet-update logic downstream.

---

## Author

Built by Shayan Waseem as part of a hands-on n8n automation learning track.
