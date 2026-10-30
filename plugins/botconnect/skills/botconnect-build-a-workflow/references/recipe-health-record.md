---
recipe: health-record
version: 1
purpose: A personal health record in the person's own memory. An overview of conditions, medications, allergies and the care team, a list of measurements, a list of visits, and a log for each month, which the person's AI fills from what it knows and keeps up to date every week.
audience:
  - role: self
    who: The person and their AIs. Nobody else reads it unless the person shares a document
folder: health
documents:
  - path: health/overview
    title: Health overview
    template: "Template: overview"
  - path: health/measurements
    title: Measurements
    template: "Template: measurements"
  - path: health/visits
    title: Visits
    template: "Template: visits"
  - path: health/log/{month}
    title: Health log
    template: "Template: log"
schedules:
  - name: weekly-pull
    cadence: weekly
    does: Saves the new facts the AI has learned about the person's health since the last pull to health/log/{month}, such as health/log/2026-10, and brings the overview, measurements and visits up to date. It never changes a line the person corrected by hand.
external:
  - kind: host-memory
    for: What this AI already knows of the person's health, such as its own memory of past chats, or an export the person gives it from a health app or a patient portal.
conventions:
  - Every measurement names its date and its unit, such as 2026-10-29, blood pressure 128/82 mmHg.
  - Each fact says where it came from, such as "from our chat on 12 October" or "from the clinic's letter".
  - A line the person corrected by hand ends with "(corrected)", and a pull never changes it.
  - A pull adds to the month's log and never deletes; a fact that stopped being true is marked as ended, with the date.
  - Nothing leaves the person's space unless they share it. Never publish these documents with memory.set_visibility unless the person asks for that document by name.
  - The record is the person's notes, not medical advice.
---

# Recipe: a personal health record

A health record in the person's own memory, under `health/`. No group, no channel and nobody else: the documents are the person's alone. The installer saves the overview, the measurements and the visits from their templates, and the record of the installation at `bc://user/me/health/recipe`. A month's log, `health/log/2026-10` for October 2026, is made from its template the first time something is saved to it.

## How the record is kept

- **Filling it the first time.** Once it is installed, offer to fill it from what you already know of the person's health, or from an export they give you. Show the person what you would save before you save it.
- **The weekly pull.** Each week, save the new facts to the month's log, then bring the overview, the measurements and the visits up to date. Read each document first and save with its revision.
- **Corrections.** When the person corrects a line, save it ending with "(corrected)". A pull never changes such a line.

## Template: overview

````markdown
# Health overview

Updated: (date)

## Conditions

- (condition): since (date), (notes)

## Medications

- (medicine), (dose), (how often): since (date), for (condition)

## Allergies

- (allergy): (reaction)

## Care team

- (name), (role), (practice): (how to reach them)
````

## Template: measurements

````markdown
# Measurements

| Date | Measurement | Value | Unit | Source |
|---|---|---|---|---|
| | | | | |
````

## Template: visits

````markdown
# Visits

## (date): (who), (where)

- Why: 
- What they said: 
- Next: 
````

## Template: log

````markdown
# Health log, (month)

- (date): (the fact), from (where it came from)
````

## Rules

Written from the recipe's manifest. **Checked by the server** means BotConnect refuses a call that breaks the rule. A **convention** is a rule the server doesn't check yet: every AI that works here keeps it, and says so when it can't.

### Checked by the server

- Your own space: the documents under `health` are yours alone, and nobody else reads them unless you publish one with memory.set_visibility.

### Conventions

- Every measurement names its date and its unit, such as 2026-10-29, blood pressure 128/82 mmHg.
- Each fact says where it came from, such as "from our chat on 12 October" or "from the clinic's letter".
- A line the person corrected by hand ends with "(corrected)", and a pull never changes it.
- A pull adds to the month's log and never deletes; a fact that stopped being true is marked as ended, with the date.
- Nothing leaves the person's space unless they share it. Never publish these documents with memory.set_visibility unless the person asks for that document by name.
- The record is the person's notes, not medical advice.
