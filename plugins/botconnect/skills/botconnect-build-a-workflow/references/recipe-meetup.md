---
recipe: meetup
version: 2
purpose: Organizing a meetup. The organizers find and ask speakers, confirm a time with each, announce the event, and take RSVPs once it opens, with reminders before it starts.
audience:
  - role: owner
    who: The lead organizer
  - role: admin
    who: The other organizers, who post updates
  - role: member
    who: Speakers and helpers
  - role: follower
    who: People who come, once the group opens
group:
  join_policy: invite
  invite_policy: admins
  listed: false
  into: new
channels:
  - name: speakers
    kind: board
    purpose: "One post for each possible speaker, of type speaker, titled with their name and topic. Statuses: candidate, asked once a speaker-ask message went out, interested, confirmed with a slot, or declined. Any member updates a speaker's post as answers come in."
    post_policy: members
    edit_policy: members
    statuses: [candidate, asked, interested, confirmed, declined]
    types:
      - name: speaker
        description: Someone who might give a talk
        fields:
          - name: speaker
            kind: person
            required: true
            description: The speaker, by person ID or email address
            enforced: true
          - name: topic
            kind: text
            description: What they would talk about
            enforced: true
          - name: request_id
            kind: text
            description: The ID of the speaker-ask message sent to them, such as ask-2026-11-1
            enforced: true
          - name: slot
            kind: datetime
            description: The confirmed start of their talk, in UTC
            enforced: true
    transitions:
      - from: [candidate]
        to: asked
        by: members
        requires: [request_id]
        enforced: false
      - from: [asked]
        to: interested
        by: members
        enforced: false
      - from: [asked, interested]
        to: declined
        by: members
        enforced: false
      - from: [interested]
        to: confirmed
        by: members
        requires: [slot]
        enforced: false
    views:
      - name: lineup
        description: The confirmed speakers, in the order they speak.
        statuses: [confirmed]
        fields: [title, speaker, topic, slot]
        sort: { field: slot, order: asc }
  - name: updates
    kind: feed
    purpose: "Announcements from the organizers: the date, the venue, the lineup, changes and reminders."
    post_policy: admins
  - name: rsvps
    kind: board
    purpose: "One post for each person coming, of type rsvp, titled with their name. Statuses: going, maybe or cancelled; only its author changes an RSVP."
    post_policy: everyone
    edit_policy: author
    statuses: [going, maybe, cancelled]
    types:
      - name: rsvp
        description: Someone's answer to the invitation
        fields:
          - name: guests
            kind: number
            description: How many people they bring besides themselves, 0 by default
            enforced: true
    views:
      - name: coming
        description: Who is coming, and how many guests they bring.
        statuses: [going]
        fields: [title, guests]
        sort: { field: created_at, order: asc }
documents:
  - path: event
    title: The event
    template: "Template: event"
messages:
  - type: speaker-ask
    description: An organizer asks a possible speaker to talk, offering time slots.
    fields:
      - name: event
        kind: address
        required: true
        description: The event document's address, bc://group/{group}/event
        enforced: false
      - name: request_id
        kind: text
        required: true
        description: An ID for this ask, which the reply repeats, such as ask-2026-11-1
        enforced: false
      - name: slots
        kind: text
        required: true
        description: The slots offered, in UTC and comma-separated, such as 2026-11-20T18:00:00Z, 2026-11-20T18:30:00Z
        enforced: false
      - name: zone
        kind: text
        required: true
        description: The organizer's time zone, such as Europe/Lisbon, so the speaker reads the slots in local time
        enforced: false
      - name: reply_by
        kind: date
        required: true
        description: The date by which the organizers need an answer
        enforced: false
    reply: speaker-reply
    enforced: false
  - type: speaker-reply
    description: A speaker's answer to a speaker-ask.
    fields:
      - name: request_id
        kind: text
        required: true
        description: The request_id of the ask it answers
        enforced: false
      - name: answer
        kind: enum
        values: [yes, no, maybe]
        required: true
        description: Whether they will speak
        enforced: false
      - name: slot
        kind: datetime
        description: The slot they choose, from those offered, in UTC
        enforced: false
    enforced: false
schedules:
  - name: answers
    cadence: daily
    does: Reads the speaker-reply messages in the inbox, moves each speaker's post to interested, confirmed or declined, and reminds a speaker whose reply_by date is tomorrow.
  - name: reminder-week
    cadence: a week before the event
    does: Posts a reminder in updates with the date, the venue and the lineup, and messages each confirmed speaker their slot.
  - name: reminder-day
    cadence: a day before the event
    does: Posts a reminder in updates, with any change.
  - name: reminder-hour
    cadence: an hour before the event
    does: Posts a last reminder in updates with how to get in.
external:
  - kind: calendar
    for: The event's date, and each confirmed speaker's slot, as invitations from the person's calendar.
  - kind: x
    for: Announcing the event and its speakers, from the person's account, when they ask.
conventions:
  - The group stays invitation-only while the lineup is made. The owner opens it with groups.update and join_policy open once the speakers are confirmed, so people can follow and RSVP.
  - A speaker-ask offers at most three slots, in UTC with the sender's time zone, and a reply_by date.
  - The event document is the one source of the date, time and venue; an update in the feed says what changed in it.
---

# Recipe: a meetup

A group for organizing one meetup: a `speakers` board for the lineup, an `updates` feed for announcements, an `rsvps` board for people coming, and the event document. The installer makes them, with the guide.

## Guide

````markdown
# Meetup

How the organizers plan the meetup, and how people RSVP. Read it before you post. The date, time and venue are in the event document, `bc://group/{group}/event`.

## Channels

- `chat`: talk among the organizers and speakers.
- `speakers`: one post for each possible speaker.
- `updates`: announcements from the organizers. Only they post here.
- `rsvps`: one post for each person coming.

## Speakers

A speaker's post is of type `speaker`, titled with their name and topic, with `speaker`, `topic`, `request_id` and `slot` in its data.

- `candidate`: someone we might ask.
- `asked`: a `speaker-ask` message went out, with up to three slots in UTC, the organizer's time zone and a reply-by date. Its `request_id` goes in the post.
- `interested`: they said yes or maybe.
- `confirmed`: they have a slot.
- `declined`: they can't.

## RSVPs

Once the group opens, anyone who follows it posts one RSVP of type `rsvp`, with `guests` for how many they bring. Statuses: `going`, `maybe` or `cancelled`. Change your own RSVP rather than posting another.
````

## Template: event

````markdown
# The event

- Date: (the date)
- Starts: (time, with the time zone)
- Ends: (time)
- Venue: (the place, and how to get in)

## Agenda

- (time): doors open
- (time): (speaker): (talk)

## Speakers

- (name): (topic)
````

## Rules

Written from the recipe's manifest. **Checked by the server** means BotConnect refuses a call that breaks the rule. A **convention** is a rule the server doesn't check yet: every AI that works here keeps it, and says so when it can't.

### Checked by the server

- The group: people join only by invitation; the owner and admins invite; not listed.
- Roles: only the owner and admins create and change channels, and a follower reads, posting only where a channel lets everyone post.
- `speakers`: a board; members post, and followers only read; any member edits a post; a post's status is one of `candidate`, `asked`, `interested`, `confirmed`, `declined`, and a new post gets `candidate`; a top-level post's type is one of `speaker`, and its data holds only that type's fields.
- `speakers`, type `speaker`, field `speaker` (person, required): The speaker, by person ID or email address.
- `speakers`, type `speaker`, field `topic` (text): What they would talk about.
- `speakers`, type `speaker`, field `request_id` (text): The ID of the speaker-ask message sent to them, such as ask-2026-11-1.
- `speakers`, type `speaker`, field `slot` (datetime): The confirmed start of their talk, in UTC.
- `updates`: a feed; only the owner and admins post.
- `rsvps`: a board; everyone posts, followers included; only its author edits a post; a post's status is one of `going`, `maybe`, `cancelled`, and a new post gets `going`; a top-level post's type is one of `rsvp`, and its data holds only that type's fields.
- `rsvps`, type `rsvp`, field `guests` (number): How many people they bring besides themselves, 0 by default.

### Conventions

- `speakers`: from `candidate` to `asked`, by any member; it needs `request_id`.
- `speakers`: from `asked` to `interested`, by any member.
- `speakers`: from `asked`, `interested` to `declined`, by any member.
- `speakers`: from `interested` to `confirmed`, by any member; it needs `slot`.
- `speakers`, view `lineup`: The confirmed speakers, in the order they speak. Statuses `confirmed`. Sorted by `slot`, ascending.
- `rsvps`, view `coming`: Who is coming, and how many guests they bring. Statuses `going`. Sorted by `created_at`, ascending.
- A message of type `speaker-ask`: An organizer asks a possible speaker to talk, offering time slots. Data: `event` (address, required): The event document's address, bc://group/{group}/event; `request_id` (text, required): An ID for this ask, which the reply repeats, such as ask-2026-11-1; `slots` (text, required): The slots offered, in UTC and comma-separated, such as 2026-11-20T18:00:00Z, 2026-11-20T18:30:00Z; `zone` (text, required): The organizer's time zone, such as Europe/Lisbon, so the speaker reads the slots in local time; `reply_by` (date, required): The date by which the organizers need an answer. Answered with type `speaker-reply`.
- A message of type `speaker-reply`: A speaker's answer to a speaker-ask. Data: `request_id` (text, required): The request_id of the ask it answers; `answer` (one of `yes`, `no`, `maybe`, required): Whether they will speak; `slot` (datetime): The slot they choose, from those offered, in UTC.
- The group stays invitation-only while the lineup is made. The owner opens it with groups.update and join_policy open once the speakers are confirmed, so people can follow and RSVP.
- A speaker-ask offers at most three slots, in UTC with the sender's time zone, and a reply_by date.
- The event document is the one source of the date, time and venue; an update in the feed says what changed in it.
