# Sharing with an audience

How to publish a document or a folder to named people, keep drafts back with a release, and see who read it. Everything in the memory skill's Publishing section holds here too.

## Share with an audience

With `"visibility": "audience"`, a document, or a folder with `under`, is readable only by the people you name, with the same care. Name them by person ID or email address; an address with no account yet reaches it once they sign up. `followers_of` gives it to a group's people, while they are in the group, and `link` makes a token, shown once, that adds whoever opens it with `memory.open_link`, for 30 days unless `expires_at` says otherwise.

Call `bc_run_write_function` with:

```json
{"function":"memory.set_visibility","input":{"under":"bc://user/me/rooms/acme","visibility":"audience","audience":{"people":["ana@example.com"]}}}
```

A later document under the folder joins it. `remove_people` takes someone out, `"visibility": "private"` ends it, and `memory.audiences` lists who it names. A reader finds everything published to them with `"scope": "shared"`:

Call `bc_run_read_function` with:

```json
{"function":"memory.search","input":{"query":"first quarter figures","scope":"shared"}}
```

## Draft and release

Readers outside the space stay on a released revision, for reading, search and its file. Later saves are drafts that only the space's people read, with `draft` and `released` marked in `memory.read`.

Call `bc_run_write_function` with:

```json
{"function":"memory.release","input":{"under":"bc://user/me/rooms/acme","summary":"First quarter figures"}}
```

Once a folder has a release, a new document in it waits for the next one, and so does a file attached since. Release first, then share, so nobody reads a draft in between.

## Who read it

Call `bc_run_read_function` with:

```json
{"function":"memory.readers","input":{"under":"bc://user/me/rooms/acme"}}
```

The owner of a space, or a group's admins and owner, see each reader from outside by person ID, the revision and the day. So reading a published document shows its publisher that you did. A group's own people aren't logged.
