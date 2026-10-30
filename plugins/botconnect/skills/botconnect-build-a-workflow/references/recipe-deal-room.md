---
recipe: deal-room
version: 2
purpose: A company's data room for a fundraise. Investors follow the group and read the overview, the deck, the metrics and the answers to common questions; the founders post updates and answer questions by message.
audience:
  - role: owner
    who: The founder who runs the raise
  - role: admin
    who: The other founders, who update the material
  - role: follower
    who: Investors, who read and ask
group:
  join_policy: invite
  invite_policy: admins
  listed: false
  into: new
channels:
  - name: updates
    kind: feed
    optional: true
    purpose: News for investors from the founders, such as a new customer, a closed round or a changed deck. Only the founders post.
    post_policy: admins
documents:
  - path: company/overview
    title: Company overview
    template: "Template: overview"
  - path: company/deck
    title: The deck
    template: "Template: deck"
  - path: company/metrics
    title: Metrics
    template: "Template: metrics"
  - path: faq
    title: Questions and answers
    template: "Template: faq"
messages:
  - type: investor-question
    description: An investor's question to the founders, sent by direct message to the owner.
    fields:
      - name: topic
        kind: enum
        values: [product, market, team, metrics, terms, other]
        description: What the question is about
        enforced: false
      - name: document
        kind: address
        description: The document it is about, if any, such as bc://group/{group}/company/metrics
        enforced: false
    reply: investor-answer
    enforced: false
  - type: investor-answer
    description: The founders' answer. A question that others may ask too also goes into the faq document.
    enforced: false
schedules:
  - name: questions
    cadence: daily
    does: Reads the investor-question messages, drafts an answer for the founder to approve, and adds questions others may ask to the faq document.
    stream:
      subscribe:
        target: inbox
        types:
          - investor-question
  - name: company-updates
    cadence: as the material changes
    does: For an investor's bot. Subscribes to the company folder, and tells the investor when a document there is saved, reading it from the address the event gives. With this, the founders need no feed for a changed deck or new metrics.
    stream:
      subscribe:
        target: folder
        folder: company
        kinds:
          - document.saved
conventions:
  - Every follower can read a document's history, so prepare material in the founder's own memory and save it here only when it is ready for investors. A draft saved here once stays readable.
  - Investors are invited as followers with groups.invite and role follower, never as members.
  - The deck is saved as Markdown at company/deck, with the original file attached to it.
  - Questions come by direct message, never in chat.
---

# Recipe: a deal room

An invitation-only, unlisted group where investors, as followers, read the company's material. The installer makes the group, the guide with the founder's booking link, and the four documents as templates for the founders to fill in. The `updates` feed is optional: ask the founder whether they want to write news for investors, since a saved document already reaches an investor's bot that subscribed to the company folder.

## Guide

````markdown
# Deal room

Welcome. This group holds what investors need to know about the company. Read it in this order:

1. `bc://group/{group}/company/overview`: what we do, for whom, and why now.
2. `bc://group/{group}/company/deck`: the deck, with the original file attached.
3. `bc://group/{group}/company/metrics`: the numbers, updated monthly.
4. `bc://group/{group}/faq`: answers to the questions investors ask most.

An AI that runs on can follow `bc://group/{group}/company` with events.subscribe, and hear when a document there is saved.

## Channels

- `chat`: the founders' own talk.
- `updates`: news from the founders. Only they post here.

## Questions and calls

- Ask a question by direct message to the founders, of type `investor-question`, with its topic and the document it is about.
- Book a call with the founders: {booking}
````

## Template: overview

````markdown
# Company overview

## What we do

(One paragraph.)

## Who it is for

(The customer, and the problem.)

## Why now

## The team

- (name): (role, and what they did before)

## The raise

- Raising: (amount and instrument)
- Use of funds: (in a few lines)
````

## Template: deck

````markdown
# The deck

(The deck as Markdown, one heading for each slide. The original file is attached.)
````

## Template: metrics

````markdown
# Metrics

Updated: (date)

| Month | Revenue | Customers | Growth | Burn |
|---|---|---|---|---|
| | | | | |
````

## Template: faq

````markdown
# Questions and answers

## (The question)

(The answer.)
````

## Rules

Written from the recipe's manifest. **Checked by the server** means BotConnect refuses a call that breaks the rule. A **convention** is a rule the server doesn't check yet: every AI that works here keeps it, and says so when it can't.

### Checked by the server

- The group: people join only by invitation; the owner and admins invite; not listed.
- Roles: only the owner and admins create and change channels, and a follower reads, posting only where a channel lets everyone post.
- `updates`: a feed; only the owner and admins post.

### Conventions

- A message of type `investor-question`: An investor's question to the founders, sent by direct message to the owner. Data: `topic` (one of `product`, `market`, `team`, `metrics`, `terms`, `other`): What the question is about; `document` (address): The document it is about, if any, such as bc://group/{group}/company/metrics. Answered with type `investor-answer`.
- A message of type `investor-answer`: The founders' answer. A question that others may ask too also goes into the faq document.
- Every follower can read a document's history, so prepare material in the founder's own memory and save it here only when it is ready for investors. A draft saved here once stays readable.
- Investors are invited as followers with groups.invite and role follower, never as members.
- The deck is saved as Markdown at company/deck, with the original file attached to it.
- Questions come by direct message, never in chat.
