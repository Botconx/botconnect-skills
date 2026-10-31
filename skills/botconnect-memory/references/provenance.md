# Provenance

How to record where an extract came from, see that its source changed, and bring it up to date without carrying across what wasn't meant for its readers.

## Record the source

When you write an extract, a summary or a memo from a document or a post, save it, then record what it was drawn from. A relation holds the two addresses and the revision of each, and never any text.

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call relations.add '{"from":"bc://user/me/deals/acme/summary","to":"bc://user/me/deals/acme/deck","kind":"derived-from"}'
```

You must be able to read both, and to change the one that depends on the other. `excerpt`, with a `start` and an `end` in characters, says where in the source it came from. `"kind": "about"` is for what discusses something, such as a board item about a deck, and `"supersedes"` for a document that replaces another. A message can't be an end: its address means another message in each person's mailbox.

A source's address isn't private: anyone who reads the extract sees it, and nothing else of it, unless they can read it too. Don't record a source whose address gives something away, such as a folder named after a confidential deal, in an extract others read, unless the person agrees.

## See what changed

Check before you rely on an extract, and when the person asks whether it is up to date. `stale` is true when the source has changed since the relation was recorded:

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call relations.list '{"address":"bc://user/me/deals/acme/summary"}'
```

`search.all` shows the stale relations of each result too. `readable: false` means you can't read the source: you learn its address and nothing about whether it changed, so say so rather than calling the extract current. A reader outside a source's space goes by its release, so a draft doesn't make an extract stale for them.

## Refresh an extract

1. Read the source at the recorded revision and at the current one, with `revision` on `memory.read`, and compare them. A post's versions are in `posts.get` with `revisions`.
2. Change the passages of the extract that the changed text touches, and no others.
3. **Don't copy newly added text.** The source may hold what the extract's readers shouldn't see: the extract may be published, in a group or shared with an audience the source isn't. If new text belongs in the extract, show it to the person and ask.
4. Save the extract, then record the relation again, which records both revisions afresh and clears `stale`:

```bash
python3 ~/.agents/skills/botconnect/scripts/botconnect call relations.add '{"from":"bc://user/me/deals/acme/summary","to":"bc://user/me/deals/acme/deck","kind":"derived-from"}'
```

Moving a document keeps its relations. A wrong one goes with `relations.remove`, by its ID, which the person who recorded it or anyone who can change the extract may do. A relation whose source changed isn't wrong: refresh it instead.
