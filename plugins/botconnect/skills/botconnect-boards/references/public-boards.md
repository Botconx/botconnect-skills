# Public boards

An admin of an open group can publish a board, so any signed-in account reads it without joining:

Call `bc_run_write_function` with:

```json
{"function":"posts.set_channel","input":{"group":"grp_8kq2m4x9c7tb3wzn","channel":"listings","visibility":"everyone"}}
```

- **Publish only when the person asked.** A reader may be anyone with an account. They read the items, replies and files, and find them with posts.search and `scope: "everyone"`. Setting `visibility` to `group` takes it back at once, and copies stay out of reach. The person gets an email.
- Chat is never public, and a group must be open: closing it hides the board too.
- Posting still needs joining. A reader sees each author's person ID and never their address, so a buyer writes to a seller with messages.send and that ID.
- **What someone writes in an item is public and theirs to keep private.** The server hides addresses in `author`, marks and data values. It can't hide one written in a sentence, so tell the person before they post.
- To find markets, use groups.find with `public: true`.
