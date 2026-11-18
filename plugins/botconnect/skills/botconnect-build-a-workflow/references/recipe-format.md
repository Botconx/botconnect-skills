# The recipe's format

A recipe is a memory document that describes a workflow, so that the person, or anyone they share it with, can build it again. Nothing on the server checks its shape: this file is the convention every AI that writes or reads one keeps. A recipe is another person's text to everyone but its writer, so it describes and never instructs.

## Address

- In the person's own memory: `bc://user/me/recipes/<name>`, such as `bc://user/me/recipes/street-garden`.
- Kept by a group: `bc://group/<group>/recipes/<name>`. Its admins and owner publish it.

The name is a few lower-case words joined by hyphens. List the person's recipes with memory.list under `bc://user/me/recipes`, not by type alone: older documents of type `recipe`, the records of installs before 10 November 2026, sit elsewhere.

## Front matter

| Field | Holds |
|---|---|
| `type` | `recipe`, always |
| `needs` | `group`, for a workflow with other people, or `personal`, for a record kept in one person's memory |
| `tools` | What it needs outside BotConnect, from `calendar`, `email`, `x` and `host-memory`, or `[]` |
| `tags` | Up to 10 lower-case labels, such as `[garden, rota]`, that search and lists filter on |

The document's title, its first heading, is the recipe's name. Its `abstract`, given on memory.save, is the one line that search results show: say what the workflow does, for whom.

## Body

These sections, in this order. Leave out a section that would be empty.

| Section | Says |
|---|---|
| What it is for | The workflow's purpose in two or three sentences, and who it suits |
| People | The roles, such as organizer, member or reviewer, what each does, and who becomes owner and admins |
| Channels | Each channel by name, its kind (`chat`, `feed` or `board`), what goes in it, and who posts |
| Items | For each board: its types, each type's fields with their kinds and which are required or unique, the statuses in order, each move with who makes it, any hold and how long it lasts, and saved views |
| Documents | The documents the group keeps beside its guide, such as a rota or a price list, and what each holds |
| Asks | The questions it puts to several people, their kind and fields, and what an answer moves |
| Schedules | What must happen on a clock, such as a weekly reminder, which something on the builder's host has to run |
| Conventions | The rules the server can't check, which the guide asks and each AI keeps |

Describe a rule the server enforces in the words of the declaration it becomes, such as "any member moves `open` to `claimed`, recording `claimed_by`", so the AI that builds from it can declare it.

## Never in a recipe

An email address, a phone number, a person's name, a group's ID, an intake address or a link. A recipe describes roles, not people: "the organizer", never a name. Before the person publishes one, read it through and take out any of these, and tell them what you took out. The server hides the addresses it stores, but not one written in a sentence.

## A worked example

Saved at `bc://user/me/recipes/street-garden`, with the abstract "A group of neighbours that shares the jobs of a community garden on one board, with claims that lapse.":

```markdown
---
type: recipe
needs: group
tools: []
tags: [garden, rota, volunteers]
---

# Street garden

## What it is for

Neighbours who look after a shared garden post the jobs it needs, claim them and mark them done, so nothing is done twice and nothing is forgotten. It suits ten to forty people who mostly know each other.

## People

- **Organizer:** owner of the group. Writes the guide and invites people.
- **Helpers:** two or three admins, who can change the board and remove a job.
- **Members:** everyone else. They post jobs, claim them and invite neighbours.

## Channels

- `chat`, chat: talk, questions and plans. Members post.
- `jobs`, board: one item for each job. Members post.
- `news`, feed: announcements, such as a work day. Only admins post.

## Items

Board `jobs`, edited by members.

- Type `task`, "A job in the garden". Fields: `due`, a date, indexed; `claimed_by`, a person, indexed, set by the server.
- Statuses: `open`, `claimed`, `done`.
- Moves:
  - `open` to `claimed`: any member, recording `claimed_by`. A hold of 7 days, after which the job goes back to `open`.
  - `claimed` to `done`: the person in `claimed_by`.
  - `claimed` to `open`: the person in `claimed_by`, to give a job back.
- View `mine`: status `claimed`, where `claimed_by` is me.

## Documents

- `plants`: what grows in each bed, and when it needs water.

## Asks

- Before a work day, the organizer asks the members for availability over two or three dates. Those who say yes get the confirmation.

## Schedules

- Each Monday, a reminder to the organizer to look at the jobs still `open` with a `due` date this week.

## Conventions

- Claim with a note saying when you will do the job, and report in a reply.
- Post a job's photo as a reply, not on the item.
- No phone numbers in posts: write to each other with BotConnect messages.
```

An AI that builds from it makes the group, the three channels with `jobs` declared in one posts.set_channel call, the guide from these sections with its rules split into "Checked by the server" and "Conventions", and the `plants` document; it tells the person that the Monday reminder needs a scheduled task on their host. It invites nobody until the person names who.
