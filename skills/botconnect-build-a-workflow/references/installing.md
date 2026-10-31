# Installing a recipe

A recipe's file has three parts. Its front matter is the **manifest**: the group, the channels with their statuses, types and fields, the documents, the messages, the schedules, the external tools and the conventions. Its body holds the **guide**, under "Guide", and each document's **template**, under the heading the manifest names, each in a fenced `markdown` block. Its **Rules** section, at the end, lists what the server checks and what is a convention.

Install it in the order below. Every step can be repeated safely, so an installation that stops halfway resumes from its record. Describe each function before its first call.

## 1. Agree it with the person

Show the person what the recipe sets up, from its manifest: the purpose, the channels and their statuses, the documents and the conventions. Then ask:

- **For a group recipe:** a name for the new group, or which of their groups to install it into. A manifest whose `group.into` is `existing` installs only into a group they run; `either` lets them choose.
- **Who to invite,** by email address, and with which role.
- **Each external tool** in `external`: whether you have it here, such as a calendar or an email account. Note each answer, such as "yes: Gmail connector" or "no".
- **Each word in braces in the guide** other than `{group}`, such as `{booking}`: what to put there. For a booking link, the person's profile may have one (profiles.get).

Build it only when they agree.

## 2. Look for an earlier installation

Every installation keeps a record, a document of type `recipe`: at `bc://group/<group>/recipe` for a group recipe, and at `bc://user/me/<folder>/recipe` for a personal one. When installing into a group the person already has, or when a group of the chosen name is in groups.list, read it:

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call memory.read '{"address":"bc://group/grp_5vh7n2c8r4kd6xpq/recipe"}'
```

- **None** (`not_found`): install from step 3.
- **`state: installing`:** an earlier session stopped partway. Use the group its front matter names, skip steps 4 and 5, and carry on from step 6. Every step below that has already been done changes nothing.
- **`state: installed` at the recipe's version:** it is done. Tell the person, and stop.
- **`state: installed` at an older version:** this is an upgrade. Run from step 6, which changes the channels to the new manifest and adds the documents that are missing, then tell the person what changed between the two manifests. The record's own front matter holds the old version's manifest.

## 3. Check there is room

- **A new group:** a person can be in 200 groups. Count theirs with groups.list.
- **An existing group:** a group has at most 20 channels, `chat` and archived ones included. List them with posts.channels, and count the recipe's channels that aren't there yet.
- **A personal recipe:** list the folder with memory.list, `under` the folder's address, to see what is already there. A person's memory holds 5,000 documents; if a save is refused with `limit_reached`, stop and tell the person.

If it doesn't fit, say what is in the way and let the person decide.

## 4. Create the group

For a group recipe with no group yet, create it from the manifest's `group` and `purpose`, with the name the person chose:

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call groups.create '{"name":"Northwind deals","purpose":"A fund'\''s deal pipeline.","join_policy":"invite","invite_policy":"admins","listed":false}'
```

Use the group's ID, such as `grp_5vh7n2c8r4kd6xpq`, wherever this page says `<group>`.

## 5. Save the first record

Save the record now, with `state: installing`, so a session that starts after an interruption finds the group and resumes:

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call memory.save '{"address":"bc://group/grp_5vh7n2c8r4kd6xpq/recipe","title":"Recipe deal-pipeline","content":"---\ntype: recipe\nrecipe: deal-pipeline\nversion: 1\nstate: installing\ngroup: grp_5vh7n2c8r4kd6xpq\nchannels: []\ndocuments: []\nexternal:\n  email: \"no\"\n---\n\n# Recipe deal-pipeline, version 1\n\nBeing installed with the build-a-workflow skill. A new session resumes from here.\n","expected_revision":0}'
```

For a personal recipe, save it at `bc://user/me/<folder>/recipe` the same way, with no `group`.

## 6. Set up each channel

A channel with `optional: true` is the person's choice: ask whether they want it, and if not, leave it out here and remove its line from the guide.

For each channel in the manifest, call posts.set_channel with its `name` as `channel`, and its `kind`, `purpose`, `post_policy`, `edit_policy` and `statuses` as the manifest gives them. Leave out what the manifest leaves out.

Send `types` too, once for each channel with a type whose fields say `enforced: true`: each such type with its `name`, `description` and `fields`, each field without its `enforced` line and with `{group}` replaced by the group's ID. Leave out a type whose fields say `enforced: false`. The server then checks every top-level post against these types:

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call posts.set_channel '{"group":"grp_5vh7n2c8r4kd6xpq","channel":"deals","kind":"board","purpose":"One post for each company we look at, of type deal.","post_policy":"members","edit_policy":"members","statuses":["proposed","investment-committee","due-diligence","term-sheet","closing","invested","rejected"],"types":[{"name":"deal","description":"A company the fund is looking at","fields":[{"name":"deal_id","kind":"text","required":true,"unique":true,"indexed":true,"description":"The deal'\''s ID, such as acme-robotics-2026"},{"name":"owner","kind":"person","required":true,"indexed":true,"description":"The team member who leads the deal"}]}]}'
```

Send `views` too, for each channel that has them: each view with its `name`, `description`, `statuses`, `type`, `where` and `sort`, without its `fields`, which only say what to show. Leave `me` and `now` as they are, because the server resolves them for each caller. Send a channel's `types` and `views` in one call, since a view can name only a declared, indexed field:

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call posts.set_channel '{"group":"grp_5vh7n2c8r4kd6xpq","channel":"deals","views":[{"name":"my-deals","description":"Your own deals still in play, soonest next action first.","statuses":["proposed","investment-committee"],"where":{"owner":"me"},"sort":{"field":"next_action_due","order":"asc"}}]}'
```

Send `transitions` too, for each channel whose rules say `enforced: true`, in the same call as its `types`, since a rule can name only a declared field. Write one rule for each status in `from`, with that status as `from`, `"*"` as `"any"`, and `to`, `by`, `requires`, `records` and `expires` as the manifest gives them, without `note` and `enforced`. Leave out rules that say `enforced: false`: the guide and the Rules section carry them as conventions. From then on a post makes only these moves, and a status no rule leaves is final:

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call posts.set_channel '{"group":"grp_5vh7n2c8r4kd6xpq","channel":"listings","transitions":[{"from":"open","to":"reserved","by":"everyone","records":"reserved_by","expires":{"after":"48h","to":"open"}},{"from":"reserved","to":"open","by":"field:reserved_by"},{"from":"reserved","to":"sold","by":"author"}]}'
```

For a channel with `intake: true`, turn on its address, in a call of its own, and for each folder in `intake_folders`, turn on its save-to address, under the group's address. A refusal with `not_allowed` means this deployment receives no email: go on without it, and say so in step 11.

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call posts.set_channel '{"group":"grp_5vh7n2c8r4kd6xpq","channel":"deals","intake":true}'
```

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call memory.set_intake '{"under":"bc://group/grp_5vh7n2c8r4kd6xpq/deals/inbox","enabled":true}'
```

Each answer gives the address. Tell the person in the chat, and never write it in the guide or the record, which every member reads: whoever has it can write to the board or the folder.

Setting a channel that exists changes it rather than making another, so this step is safe to repeat. In an upgrade, a unique field can be refused with `conflict` when two existing posts share a value: show the person the two posts and let them change one. An existing post that doesn't fit stays listed, and must fit the next time it is edited.

## 7. Save the guide

The guide is the "Guide" block of the recipe, with `{group}` replaced by the group's ID and each other word in braces by the person's answer, followed by a blank line and the recipe's whole Rules section. Save it at the group's guide address, as a new document:

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call memory.save '{"address":"bc://group/grp_5vh7n2c8r4kd6xpq/guide","title":"Guide","content":"# Deal pipeline\n\nHow this group tracks the companies the fund looks at.\n\n## Rules\n\n…","abstract":"A fund'\''s deal pipeline.","expected_revision":0}'
```

If the group already has a guide, the save fails with `conflict` and returns it. Read it with the person, merge the recipe's text into it, and save with its revision.

## 8. Save the documents

For each document in the manifest whose path has no braces, save the block under its template heading at its path: `bc://group/<group>/<path>` in a group, `bc://user/me/<path>` for a personal recipe, with `expected_revision` 0. A `conflict` means the document is already there: leave it as it is. A path with braces, such as `deals/{deal_id}/memo` or `health/log/{month}`, is made later, from the same template, when the first deal or the first month's facts arrive.

## 9. Save the record again

Now save the record with `state: installed`, and the record's current revision as `expected_revision`: 1 when step 5 made it, or the revision you read in step 2. Its front matter lists what was made and the answers about external tools:

```yaml
type: recipe
recipe: deal-pipeline
version: 1
state: installed
group: grp_5vh7n2c8r4kd6xpq
channels: [deals]
documents:
  - bc://group/grp_5vh7n2c8r4kd6xpq/guide
  - bc://group/grp_5vh7n2c8r4kd6xpq/pipeline-week
external:
  email: "no"
```

Its body holds the schedules for the host, the external tools, the Rules section, and the manifest itself in a fenced `yaml` block, so a later session can compare it with a newer version of the recipe.

## 10. Invite people

Invite the people the person named, with their roles, as the skill's "Try it, then invite people" says, and offer to write the invitation emails. A personal recipe invites nobody.

## 11. Tell the person

- **Each intake address,** and that the board or folder takes email from anyone who has it, so it stays with the people who run the workflow.
- **The conventions in force:** the "Conventions" list of the Rules section. Their AIs keep them; the server doesn't check them yet.
- **The schedules** in the manifest, each with its cadence and what it does. Set up the ones this host can run, such as a scheduled task, when the person agrees, and say which it can't. A schedule with `stream` can run on the event stream instead, on a host with a bot that keeps running under a consumer name: subscribe as its `subscribe` says, set reminders as its `remind` says, and read as "React instead of polling" in the skill says.
- **The external tools,** with what you have and what is missing. BotConnect holds no credential for any of them.
