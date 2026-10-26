# The profile template

A person's profile is one document in their memory, at `bc://user/me/profile`, that says who they are and what they do, for other people and their AIs to read once it is published. Write it from what the person tells you, in their words, and show it to them before it is published.

`profiles.save` takes the template's fields below, writes them as this frontmatter, and saves the profile at that address; `about` is the body. The frontmatter, between the `---` lines, holds the fields an AI compares and searches. The body, after it, is prose. Who can read the profile is set with `profiles.publish`, never in the frontmatter.

- `type: profile` stays as it is. It is what a search for profiles matches.
- `name` and `headline` are required: the name the person goes by, and one line on what they do. Everything else is optional: leave out a field the person didn't give, rather than guessing.
- `location` is a city or region, as precise as the person wants it public.
- `skills` is a short list, the words people would search for.
- `experience` and `education` are lists, newest first. Dates are `YYYY-MM`, such as `2024-03`, so AIs can compare them. `end: null` means it continues now.
- `links` are pages the person wants to share, each with a label and its full address as they gave it.
- Contact details, such as an email address or a phone number, go in only if the person asks for them to be public. Other people can already write to them by their person ID.
- No photo: a profile is text.

```markdown
---
type: profile
name: Ana Lopez
headline: Product designer, health software
location: Lisbon
skills: [Product design, Accessibility, Figma]
experience:
  - title: Product designer
    organization: Example Health
    start: 2024-03
    end: null
education:
  - school: Example University
    degree: BA
    field: Design
    start: 2016-09
    end: 2020-06
links:
  - label: Portfolio
    url: example.com/ana-lopez
---

# About

A few paragraphs, in the person's words: what they do, what they are looking for, and what they would like to be asked about.
```

Give the save an `abstract` of one or two sentences, such as the headline and the location: search results and lists show it.
