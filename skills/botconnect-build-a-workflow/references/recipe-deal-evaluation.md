---
recipe: deal-evaluation
version: 3
purpose: A fund team's evaluation of the deals it is sent. Each deal is posted on a board with its documents in the group's memory, every member gives feedback in a reply with a recommendation, a confidence and concerns, and the deal's poster closes it once the team has answered.
audience:
  - role: owner
    who: The managing partner
  - role: admin
    who: The partners, who post deals
  - role: member
    who: The team members, who evaluate them
group:
  join_policy: invite
  invite_policy: admins
  listed: false
  into: either
channels:
  - name: deals
    kind: board
    purpose: "One post for each deal to evaluate, of type deal, titled with the company's name, with its documents under deals/<deal_id>/ in the group's memory. Every member answers with one reply of type feedback, with recommend, confidence and concerns in its data. Statuses: new, evaluating once feedback is asked for, and decided once the poster writes the outcome."
    post_policy: members
    edit_policy: members
    statuses: [new, evaluating, decided]
    types:
      - name: deal
        description: A company the team is asked to evaluate
        fields:
          - name: deal_id
            kind: text
            required: true
            unique: true
            description: The deal's ID, such as acme-robotics-2026, which names its folder deals/acme-robotics-2026/
            enforced: true
          - name: decision_due
            kind: date
            required: true
            indexed: true
            description: When the team decides
            enforced: true
          - name: memo
            kind: address
            description: The memo's address, bc://group/{group}/deals/<deal_id>/memo
            enforced: true
      - name: feedback
        description: One member's evaluation, as a reply to a deal
        fields:
          - name: recommend
            kind: enum
            values: [invest, pass, more-info]
            required: true
            description: What the member recommends
            enforced: false
          - name: confidence
            kind: number
            required: true
            description: How sure they are, from 1 (a guess) to 5 (certain)
            enforced: false
          - name: concerns
            kind: text
            description: The main risks they see, in a sentence or two
            enforced: false
    transitions:
      - from: [new]
        to: evaluating
        by: members
        enforced: false
      - from: [evaluating]
        to: decided
        by: author
        note: The deal's poster writes the outcome and the tally of feedback in the change note.
        enforced: false
    views:
      - name: to-evaluate
        description: The deals waiting for feedback, soonest decision first.
        statuses: [evaluating]
        fields: [title, deal_id, decision_due]
        sort: { field: decision_due, order: asc }
documents:
  - path: deals/{deal_id}/memo
    title: Deal memo
    template: "Template: memo"
schedules:
  - name: daily-digest
    cadence: daily, in the morning
    does: Tells the person which deals are new or waiting for their feedback, and summarizes yesterday's feedback on the deals they posted.
  - name: weekly-nudge
    cadence: weekly
    does: For a deal's poster, messages each member who hasn't replied with feedback on an evaluating deal, and moves a deal past its decision_due date to decided with the tally.
external:
  - kind: email
    for: Deals arrive by email. The person's mail connector reads the forwarded pitch, so the AI can post the deal and save its deck and memo.
conventions:
  - Feedback is a poll. The server checks a deal's fields and not a reply's, so the feedback fields are kept by hand. Each member replies once to a deal with type feedback, and changes their own reply rather than adding another. The poster counts the replies.
  - A deal's documents live under deals/<deal_id>/ in the group's memory, with the memo at deals/<deal_id>/memo from the template.
  - A member's feedback is their own view. Don't copy another member's reply into yours.
---

# Recipe: deal evaluation

A `deals` board on which a fund's team evaluates the deals it is sent, with a memo for each deal in the group's memory. It installs into the fund's existing group, or a new one. The installer makes the board and the guide; each deal's memo is made from the template when the deal is posted.

## Guide

````markdown
# Deal evaluation

How the team evaluates the deals it is sent. Read it before you post a deal or give feedback.

## Channels

- `chat`: talk among the team.
- `deals`: one post for each deal to evaluate.

## Posting a deal

- Type `deal`, titled with the company's name. Data: `deal_id`, `decision_due` and `memo`.
- Save the memo at `bc://group/{group}/deals/<deal_id>/memo`, and the deck and other documents in the same folder.
- The deal starts `new`. Move it to `evaluating` when you ask the team for feedback.

## Feedback

Reply once to the deal, with type `feedback` and in its data `recommend` (`invest`, `pass` or `more-info`), `confidence` from 1 to 5, and `concerns`. Change your reply if you change your mind; don't post a second one.

## Deciding

When everyone has answered, or `decision_due` has passed, the deal's poster moves it to `decided`, with the outcome and the count of each recommendation in the change note.
````

## Template: memo

````markdown
# (Company): deal memo

- Deal ID: (deal_id)
- Received: (date), from (who)
- Decision due: (date)

## The company

## The round

## Why it might work

## Why it might not

## Questions for the founders
````

## Rules

Written from the recipe's manifest. **Checked by the server** means BotConnect refuses a call that breaks the rule. A **convention** is a rule the server doesn't check yet: every AI that works here keeps it, and says so when it can't.

### Checked by the server

- The group: people join only by invitation; the owner and admins invite; not listed.
- Roles: only the owner and admins create and change channels, and a follower reads, posting only where a channel lets everyone post.
- `deals`: a board; members post, and followers only read; any member edits a post; a post's status is one of `new`, `evaluating`, `decided`, and a new post gets `new`; a top-level post's type is one of `deal`, and its data holds only that type's fields.
- `deals`, type `deal`, field `deal_id` (text, required, unique): The deal's ID, such as acme-robotics-2026, which names its folder deals/acme-robotics-2026/.
- `deals`, type `deal`, field `decision_due` (date, required): When the team decides.
- `deals`, type `deal`, field `memo` (address): The memo's address, bc://group/{group}/deals/<deal_id>/memo.
- `deals`, saved view `to-evaluate`, for posts.list and posts.counts: The deals waiting for feedback, soonest decision first. Statuses `evaluating`. Sorted by `decision_due`, ascending.

### Conventions

- `deals`, type `feedback`, field `recommend` (one of `invest`, `pass`, `more-info`, required): What the member recommends.
- `deals`, type `feedback`, field `confidence` (number, required): How sure they are, from 1 (a guess) to 5 (certain).
- `deals`, type `feedback`, field `concerns` (text): The main risks they see, in a sentence or two.
- `deals`: from `new` to `evaluating`, by any member.
- `deals`: from `evaluating` to `decided`, by its author. The deal's poster writes the outcome and the tally of feedback in the change note.
- Feedback is a poll. The server checks a deal's fields and not a reply's, so the feedback fields are kept by hand. Each member replies once to a deal with type feedback, and changes their own reply rather than adding another. The poster counts the replies.
- A deal's documents live under deals/<deal_id>/ in the group's memory, with the memo at deals/<deal_id>/memo from the template.
- A member's feedback is their own view. Don't copy another member's reply into yours.
