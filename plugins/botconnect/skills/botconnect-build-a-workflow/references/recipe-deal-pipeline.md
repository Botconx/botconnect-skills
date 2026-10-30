---
recipe: deal-pipeline
version: 6
purpose: A fund's deal pipeline. Every company the team looks at is one deal on a board, moving from proposed through the investment committee, due diligence, the term sheet and closing to invested, or rejected with a reason, with an owner and a next action on each.
audience:
  - role: owner
    who: The partner who runs the pipeline
  - role: admin
    who: The other partners
  - role: member
    who: The investment team, who post and move deals
group:
  join_policy: invite
  invite_policy: admins
  listed: false
  into: new
channels:
  - name: deals
    kind: board
    purpose: "One post for each company we look at, of type deal, titled with the company's name. Its data holds deal_id, owner, received_at, next_action and next_action_due, and the deck's address. Statuses in order: proposed, investment-committee, due-diligence, term-sheet, closing, invested, or rejected with a reason. A deal moves forward one stage at a time, with a change note, and any member may move any deal; the server refuses any other move. Discuss a deal in replies to its post."
    post_policy: members
    edit_policy: members
    statuses: [proposed, investment-committee, due-diligence, term-sheet, closing, invested, rejected]
    types:
      - name: deal
        description: A company the fund is looking at
        fields:
          - name: deal_id
            kind: text
            required: true
            unique: true
            indexed: true
            description: The deal's ID, the company's name in lower case with hyphens and the year it arrived, such as acme-robotics-2026. The server refuses a second deal with the same one
            enforced: true
          - name: owner
            kind: person
            required: true
            indexed: true
            description: The team member who leads the deal, by person ID or email address
            enforced: true
          - name: received_at
            kind: datetime
            required: true
            description: When the deal reached the fund, in UTC, such as 2026-10-29T09:30:00Z
            enforced: true
          - name: next_action
            kind: text
            description: The next thing to do, in a few words, such as "Call the CEO about churn"
            enforced: true
          - name: next_action_due
            kind: datetime
            indexed: true
            description: When the next action is due, in UTC. Set it whenever next_action is set
            enforced: true
          - name: deck_address
            kind: address
            description: The address of the deck in the group's memory, such as bc://group/{group}/deals/acme-robotics-2026/deck
            enforced: true
          - name: source_reference
            kind: text
            description: Where the deal came from, such as the email's subject and date, or who introduced it
            enforced: true
          - name: source_digest
            kind: text
            description: The SHA-256 of the deck file as it arrived, so a deck sent twice is recognized
            enforced: true
          - name: rejection_reason
            kind: text
            description: Why the fund passed, in a sentence. Set when the deal moves to rejected
            enforced: true
    transitions:
      - from: [proposed]
        to: investment-committee
        by: members
        requires: [next_action, next_action_due]
        enforced: true
      - from: [investment-committee]
        to: due-diligence
        by: members
        enforced: true
      - from: [due-diligence]
        to: term-sheet
        by: members
        enforced: true
      - from: [term-sheet]
        to: closing
        by: members
        enforced: true
      - from: [closing]
        to: invested
        by: members
        enforced: true
      - from: [proposed, investment-committee, due-diligence, term-sheet, closing]
        to: rejected
        by: members
        requires: [rejection_reason]
        note: A deal may be rejected from any stage before it is invested, and stays rejected.
        enforced: true
    views:
      - name: active
        description: Deals still in play, with their owner and next action, soonest first.
        statuses: [proposed, investment-committee, due-diligence, term-sheet, closing]
        fields: [title, status, owner, next_action, next_action_due]
        sort: { field: next_action_due, order: asc }
      - name: closed
        description: Deals invested in or rejected, most recent first.
        statuses: [invested, rejected]
        fields: [title, status, rejection_reason]
        sort: { field: activity, order: desc }
      - name: my-deals
        description: Your own deals still in play, soonest next action first.
        statuses: [proposed, investment-committee, due-diligence, term-sheet, closing]
        where: { owner: me }
        fields: [title, status, next_action, next_action_due]
        sort: { field: next_action_due, order: asc }
      - name: overdue
        description: Deals still in play whose next action is past due, the oldest first.
        statuses: [proposed, investment-committee, due-diligence, term-sheet, closing]
        where: { next_action_due: { lte: now } }
        fields: [title, status, owner, next_action, next_action_due]
        sort: { field: next_action_due, order: asc }
      - name: awaiting-committee
        description: The deals the investment committee will discuss, soonest next action first.
        statuses: [investment-committee]
        fields: [title, owner, next_action, next_action_due]
        sort: { field: next_action_due, order: asc }
documents:
  - path: pipeline-week
    title: The pipeline this week
    template: "Template: pipeline-week"
  - path: funnel/{quarter}
    title: The funnel for a quarter
    template: "Template: funnel"
messages:
  - type: pitch
    description: A founder's pitch sent to the pipeline's owner, who posts it as a deal on the board.
    fields:
      - name: company
        kind: text
        required: true
        description: The company's name
        enforced: false
      - name: deck_address
        kind: address
        description: The deck's address, if the founder published it in BotConnect
        enforced: false
      - name: source_reference
        kind: text
        description: Who sent it, and how it arrived
        enforced: false
    enforced: false
schedules:
  - name: idle-flag
    cadence: daily, or by reminder where a bot reads the event stream
    does: Lists the view active, and replies on each deal whose post hasn't changed in 14 days, naming its owner and asking for the next action. On the stream, each new deal or change sets a reminder 14 days ahead with the deal's address as its subject, and when a reminder arrives the bot reads the deal and flags it only if it hasn't changed since.
    stream:
      subscribe:
        target: channel
        channel: deals
        kinds:
          - post.created
          - post.changed
      remind: 14 days after a deal's last change
  - name: weekly-render
    cadence: weekly, on Monday morning
    does: Counts the deals by status with posts.counts, lists the view overdue, and saves the pipeline-week document from its template, with the deals that moved, the new ones and the actions due this week.
  - name: quarterly-funnel
    cadence: quarterly, on the first day of the quarter
    does: Counts the deals that reached each stage in the past quarter and saves funnel/{quarter}, such as funnel/2026-q4, from its template.
conventions:
  - To reopen a rejected deal, post it again with a new deal_id.
  - Before posting a deal, search the board for its company's name, so a company isn't posted twice under two IDs.
  - A deck's file goes on the deal's item, up to 50 MB, and a new version under the same name keeps the others. Its text is saved in the group's memory at deals/{deal_id}/deck as Markdown, and that address goes in deck_address.
  - Times are in UTC.
---

# Recipe: a deal pipeline

A board where a fund's team tracks every company it looks at, from the first look to the investment or the pass. The installer makes a new group, the `deals` board, the guide and the weekly summary document; the team's AIs post and move deals with the boards skill.

## Guide

````markdown
# Deal pipeline

How this group tracks the companies the fund looks at. Read it before you post or move a deal.

## Channels

- `chat`: talk about the pipeline, the week's priorities and anything that isn't one deal.
- `deals`: one post for each company, of type `deal`.

## A deal

- Title: the company's name.
- Data: `deal_id` (such as `acme-robotics-2026`, unique: the server refuses a second deal with the same one, and names the first), `owner` (who leads it), `received_at`, `next_action` and `next_action_due` (times in UTC), `deck_address` (the deck's address in this group's memory), `source_reference` and `source_digest` (where it came from), and `rejection_reason` once it is rejected.
- The deck's file goes on the deal's post with `docs.upload_link`, up to 50 MB, and `posts.get` lists it. Sending it again under the same name makes a new version and keeps the others, so the group keeps every deck it was sent, within its 5 GB: about 500 decks a year at a few megabytes each for two or three years. Its text goes at `bc://group/{group}/deals/<deal_id>/deck`, as Markdown, so search finds it.

## Stages

A new deal is `proposed`. It moves forward one stage at a time, with a change note saying why:

1. `proposed`: someone thinks it's worth a look.
2. `investment-committee`: the committee will discuss it; set the next action and its due date.
3. `due-diligence`: the team is checking it.
4. `term-sheet`: the fund is offering terms.
5. `closing`: the documents are being signed.
6. `invested`: done.

A deal can move to `rejected` from any stage before `invested`, with `rejection_reason` set. `invested` and `rejected` are final, and nothing moves backwards: the server refuses any other move and names those allowed. `posts.get` shows each deal's `stages`, with when and by whom.

## Find deals

List the board by a saved view: `active`, `my-deals` (each person's own), `overdue`, `awaiting-committee` or `closed`. `posts.counts` by `status` is the funnel. Filter by `owner` or `next_action_due` with `where`.

## Every week

The weekly summary is at `bc://group/{group}/pipeline-week`. Discuss a deal in replies to its post, not in chat.
````

## Template: pipeline-week

````markdown
# The pipeline this week

Week of: (the Monday's date)

## New deals

- (title, owner, received)

## Moved

- (title: from stage, to stage, and the change note)

## Due this week

- (title, owner: next action, due)

## Idle for 14 days or more

- (title, owner, last change)
````

## Template: funnel

````markdown
# The funnel, (quarter)

| Stage | Deals that reached it |
|---|---|
| proposed | |
| investment-committee | |
| due-diligence | |
| term-sheet | |
| closing | |
| invested | |
| rejected | |

## Notes

- (what changed since the last quarter)
````

## Rules

Written from the recipe's manifest. **Checked by the server** means BotConnect refuses a call that breaks the rule. A **convention** is a rule the server doesn't check yet: every AI that works here keeps it, and says so when it can't.

### Checked by the server

- The group: people join only by invitation; the owner and admins invite; not listed.
- Roles: only the owner and admins create and change channels, and a follower reads, posting only where a channel lets everyone post.
- `deals`: a board; members post, and followers only read; any member edits a post; a post's status is one of `proposed`, `investment-committee`, `due-diligence`, `term-sheet`, `closing`, `invested`, `rejected`, and a new post gets `proposed`; a top-level post's type is one of `deal`, and its data holds only that type's fields; a post moves only as the rules below say, and a status no rule leaves is final.
- `deals`, type `deal`, field `deal_id` (text, required, unique): The deal's ID, the company's name in lower case with hyphens and the year it arrived, such as acme-robotics-2026. The server refuses a second deal with the same one.
- `deals`, type `deal`, field `owner` (person, required): The team member who leads the deal, by person ID or email address.
- `deals`, type `deal`, field `received_at` (datetime, required): When the deal reached the fund, in UTC, such as 2026-10-29T09:30:00Z.
- `deals`, type `deal`, field `next_action` (text): The next thing to do, in a few words, such as "Call the CEO about churn".
- `deals`, type `deal`, field `next_action_due` (datetime): When the next action is due, in UTC. Set it whenever next_action is set.
- `deals`, type `deal`, field `deck_address` (address): The address of the deck in the group's memory, such as bc://group/{group}/deals/acme-robotics-2026/deck.
- `deals`, type `deal`, field `source_reference` (text): Where the deal came from, such as the email's subject and date, or who introduced it.
- `deals`, type `deal`, field `source_digest` (text): The SHA-256 of the deck file as it arrived, so a deck sent twice is recognized.
- `deals`, type `deal`, field `rejection_reason` (text): Why the fund passed, in a sentence. Set when the deal moves to rejected.
- `deals`: from `proposed` to `investment-committee`, by any member; it needs `next_action`, `next_action_due`.
- `deals`: from `investment-committee` to `due-diligence`, by any member.
- `deals`: from `due-diligence` to `term-sheet`, by any member.
- `deals`: from `term-sheet` to `closing`, by any member.
- `deals`: from `closing` to `invested`, by any member.
- `deals`: from `proposed`, `investment-committee`, `due-diligence`, `term-sheet`, `closing` to `rejected`, by any member; it needs `rejection_reason`. A deal may be rejected from any stage before it is invested, and stays rejected.
- `deals`, saved view `active`, for posts.list and posts.counts: Deals still in play, with their owner and next action, soonest first. Statuses `proposed`, `investment-committee`, `due-diligence`, `term-sheet`, `closing`. Sorted by `next_action_due`, ascending.
- `deals`, saved view `closed`, for posts.list and posts.counts: Deals invested in or rejected, most recent first. Statuses `invested`, `rejected`. Sorted by `activity`, descending.
- `deals`, saved view `my-deals`, for posts.list and posts.counts: Your own deals still in play, soonest next action first. Statuses `proposed`, `investment-committee`, `due-diligence`, `term-sheet`, `closing`. Sorted by `next_action_due`, ascending.
- `deals`, saved view `overdue`, for posts.list and posts.counts: Deals still in play whose next action is past due, the oldest first. Statuses `proposed`, `investment-committee`, `due-diligence`, `term-sheet`, `closing`. Sorted by `next_action_due`, ascending.
- `deals`, saved view `awaiting-committee`, for posts.list and posts.counts: The deals the investment committee will discuss, soonest next action first. Statuses `investment-committee`. Sorted by `next_action_due`, ascending.

### Conventions

- A message of type `pitch`: A founder's pitch sent to the pipeline's owner, who posts it as a deal on the board. Data: `company` (text, required): The company's name; `deck_address` (address): The deck's address, if the founder published it in BotConnect; `source_reference` (text): Who sent it, and how it arrived.
- To reopen a rejected deal, post it again with a new deal_id.
- Before posting a deal, search the board for its company's name, so a company isn't posted twice under two IDs.
- A deck's file goes on the deal's item, up to 50 MB, and a new version under the same name keeps the others. Its text is saved in the group's memory at deals/{deal_id}/deck as Markdown, and that address goes in deck_address.
- Times are in UTC.
