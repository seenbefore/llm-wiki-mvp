# Aliases - Obsidian Help

Source: https://help.obsidian.md/aliases
Retrieved: 2026-04-07

## Key points

- An alias is an alternative name for a note.
- Aliases should be stored in YAML frontmatter as a list.
- Example:

```yaml
---
aliases:
  - Doggo
  - Woofer
  - Yapper
---
```

- If you want to link to a note using an alias, Obsidian suggests the alias and generates:
  - `[[Artificial Intelligence|AI]]`
- Obsidian explicitly prefers this format over writing `[[AI]]` directly.
- The reason is interoperability with other applications using Wikilinks.
- If you only want to change how a link looks in one place, use custom display text instead of aliases.

## Notes

- Aliases are useful for acronyms, nicknames, and multilingual names.
- Backlinks can help find unlinked mentions of aliases.
