# Building the list

How to build the person's contact list from their own data: the sources, how to send each, and the merge that joins them into one list. A person does this about once a month; the skill's body covers the rest.

Each place the person keeps people, such as LinkedIn, Google Contacts, who they follow on X, their mailbox, their calendar or a phone's address book, becomes one **source** in BotConnect, sent by whichever of their AIs can reach it. BotConnect keeps one current version of each, and merges them all into one list without duplicates: the same person from LinkedIn and Google is one contact with both. Sources from every AI of the person land in one place, so you never need another AI's files. Send what you are good at: a mailbox, a calendar or Google Contacts when you have that connector, who they follow on X when you read X, and otherwise an export file the person downloads. Nobody is told they are in a source or on the list.

## Export files

Tell the person how to get the file. Never sign in to a site for them or scrape it.

- **LinkedIn** gives nobody their connections through its API. In LinkedIn's settings, under data privacy, the person asks for a copy of their data, with connections, or the whole archive, which also counts their LinkedIn messages. LinkedIn emails them a zip within a day. Send the zip as it is, or its `Connections.csv`.
- **Google Contacts**: on contacts.google.com, export, as a Google CSV. A phone's address book syncs with it.
- **A vCard file** (`.vcf`), from a phone or a mail app.
- **X**: the accounts they follow, as a CSV from an export tool, with each account's numeric ID.

Send the file, zipped first when it is large. BotConnect reads it at once, says how many people it found, and refuses a file it can't read:

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call contacts.source_file '{"name":"Connections.csv"}' --file content_base64=Connections.csv
```

## Rows from a connector

What you read from a connector rather than a file, you send as rows, up to 200 a call: the first call starts a draft, later ones add to it, and `complete: true` makes it the source's current version. Give each row every name, address, phone and handle you have, and a `record_id` that stays the same each time, such as Google Contacts' `resourceName`, when the connector has one. Who the person follows on X is `"kind": "x"`, with each account's `x` handle and `x_id`. Anything else, such as a CRM's people, is `"kind": "other"` with a `label`:

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call contacts.source_rows '{"kind":"other","label":"CRM","start":true,"rows":[{"name":"Ana Lopez","emails":["ana@example.com"]}],"complete":true}'
```

### A mailbox

Take the people the person wrote to or heard back from in the last year. Search the mail by its senders, recipients and dates only, without the bodies, a page at a time, and take:

- everyone in `To` or `Cc` of a message the person sent, whether they answered or not
- everyone who wrote to the person in a thread the person answered

Leave out mailing lists, newsletters, notifications, receipts, the person's own addresses, addresses such as `no-reply@` or `info@`, and a mail service's bounce, unsubscribe and tracking addresses. Send one row per address. Name it from the mail's display names, or a `first.last@` address. Its note counts the messages the person sent to it and received from it. `last_contact_at` is the last message either way, and `last_heard_at` the last message from that address, left out when it never wrote. A contact's primary address becomes the one they last wrote from:

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call contacts.source_rows '{"kind":"email","start":true,"rows":[{"name":"Ana Lopez","emails":["ana@example.com"],"notes":"14 sent, 9 received","last_contact_at":"2026-10-20T09:30:00Z","last_heard_at":"2026-10-18T16:05:00Z"}],"complete":true}'
```

### A calendar

From a calendar connector, read the events of the last year that already happened, a month at a time, keeping only each event's date, title and attendees. Skip events the person declined or with more than 20 attendees. Take each attendee except the person, attendees who declined, rooms and resources, and group addresses. An attendee's name is their display name; without one, the name the mailbox gives that address, or a `first.last@` address; otherwise leave it empty. Send one row per address. Each occurrence of a repeating meeting is one meeting: `last_contact_at` is the latest, and the note says how many there were and the last one's title. The merge asks whether to keep an attendee with no name whose address is on no contact:

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call contacts.source_rows '{"kind":"calendar","start":true,"rows":[{"name":"Ana Lopez","emails":["ana@example.com"],"notes":"5 meetings, the last: Pricing review","last_contact_at":"2026-10-28T15:00:00Z"}],"complete":true}'
```

## Merge the sources

List the sources. When one isn't merged yet, ask the person whether more sources are coming before you preview a merge, and preview once they are all in: each preview reads every current source, and the person has 10 a day. The context's `contacts` section names the sources not merged yet too:

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call contacts.sources
```

Preview the merge. It reads every current source and the list as it is, and is usually ready in under 30 seconds, so read it after 10:

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call contacts.merge_preview
```

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call contacts.merge_get
```

Show the person what it would do: the new contacts, the contacts it updates, records gone from their source, whose contact stays, and records it leaves out because they can only be reached at a role address such as `info@`, with a few of each. Then answer its questions yourself, from each question's `a`, `b`, `reason` and `suggested`:

- **Same person?** Most records that share a name are the same person, so answer `same` unless they clearly describe different people, such as two different LinkedIn profiles, or different careers in different places. When unsure, answer `same`.
- **Keep this contact?** A record with no name but something more, such as a phone: keep it when it looks like someone the person knows.

Ask the person only about the few you can't judge, at most ten, and tell them how you answered the rest. Send many answers in one call. Every later merge follows them:

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call contacts.answer '{"answers":[{"question":"q3f9a1c2b7e","answer":"same"}]}'
```

Preview again so the answers count, and once the person approves, apply it with `contacts.merge_apply` and the preview's `id`, such as `mrg_8kq2m4x9c7tb3wzn`. A large merge takes a few minutes, which `contacts.merge_get` shows. People whose questions are still open stay out until they are answered. A field the person or an AI set with `contacts.update` is never changed by a merge.

Tell the person what changed, then offer to fill in the list from the web with `contacts.enrich`, as the skill's section "Fill in contacts from the web" says. When a merge joined two people wrongly, `contacts.get` lists the `records` a contact holds, such as `linkedin:sam-rivera-42`, and `contacts.split` with the contact and those record IDs takes them out into a contact of their own, for good.

## Limits

| Limit | Value |
|---|---|
| Sources | 10, one of each kind, and up to 5 others |
| People in a source | 50,000 |
| Rows in a call of `contacts.source_rows` | 200 |
| A file | 3 MB as base64, or 4 MB by link or as a ChatGPT file, zipped or not, and 10 MB unzipped |
| Files sent, and drafts completed | 20 a day, or 10 for a new account |
| Merges previewed | 10 a day, or 5 for a new account |
