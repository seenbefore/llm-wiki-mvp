# Internal links - Obsidian Help

Source: https://help.obsidian.md/links
Retrieved: 2026-04-07

## Key points

- Obsidian supports both Wikilink and Markdown internal link formats.
- Wikilink examples: `[[Three laws of motion]]` and `[[Three laws of motion.md]]`
- Markdown examples: `[Three laws of motion](Three%20laws%20of%20motion)` and `[Three laws of motion](Three%20laws%20of%20motion.md)`
- By default, Obsidian generates Wikilinks.
- When using Markdown links, the destination should be URL encoded.
- Some characters may not work reliably inside link destinations, including `# | ^ : %% [[ ]]`.
- To customize how a link appears, use display text:
  - Wikilink: `[[Example|Custom name]]`
  - Markdown: `[Custom name](Example.md)`
- For one-off display changes, use link display text.
- For repeated alternate names, use aliases instead.

## Headings and blocks

- Link to headings with `[[Note#Heading]]`
- Link to blocks with `[[Note#^block-id]]`

## Notes

- Obsidian can update internal links when files are renamed.
- Links to non-Markdown files should include the extension, such as `[[Figure 1.png]]`.
