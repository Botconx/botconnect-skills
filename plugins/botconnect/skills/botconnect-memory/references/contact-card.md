# The contact card template

A contact card is one document in the person's memory, at `bc://user/me/contact-card`, that holds what they give their contacts and not the world: a phone number, a booking link, what they are asking for and offering now. It is published to contacts, so only the people on their contact list read it. Their public profile, at `bc://user/me/profile`, keeps the rest.

Fill in the template below from what the person tells you, in their words, and show it to them before it is published.

- `type: contact-card` stays as it is. It is what a search for contact cards matches.
- `name` is required. Everything else is optional: leave out a field the person didn't give, rather than guessing.
- `phone` and `booking` are only for people the person chose. Put them here, never in the public profile, unless the person asks.
- `asks` and `offers` are short lists, each one line: what the person is looking for now, such as an introduction or a hire, and what they can help with. Keep them current, because contacts' AIs search them.
- `updated` is the date the asks and offers were last checked, as `YYYY-MM-DD`.

```markdown
---
type: contact-card
name: Ana Lopez
phone: "+351 912 345 678"
booking: example.com/ana/30-minutes
asks:
  - An introduction to a health insurer's product team
  - A front-end developer for three months
offers:
  - Reviews of accessibility in a product's design
updated: 2026-10-25
---

# For my contacts

A few lines, in the person's words: how they like to be reached, and what they are working on now.
```

Give the save an `abstract` of one sentence, such as the first ask and the first offer: search results and lists show it. Then publish it to contacts with `memory.set_visibility` and `"visibility": "contacts"`.
