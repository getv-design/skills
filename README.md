# vDesign skills

The skill that teaches an AI agent to write [vDesign](https://getv.design) documents:
posters, diagrams, slide decks, illustrations, web pages and any other visual design, in the `.vd`
format. A `.vd` keeps sizes, spacing and colours as named relationships rather than fixed
coordinates, so a design an agent made stays easy to change afterwards, by you or by the
agent.

The skill follows the open [Agent Skills](https://agentskills.io) format, so any agent that
loads skills can use it. This repository is also a Claude Code plugin marketplace, and the
plugin adds the vDesign connector beside the skill.

```
skills/vdesign/            the skill: SKILL.md, the reference, and example documents
.claude-plugin/            the Claude Code plugin and its marketplace
.mcp.json                  the vDesign connector, https://mcp.getv.design/mcp
```

## Install

**Claude Code**, the skill and the connector together:

```
/plugin marketplace add getv-design/skills
/plugin install vdesign@getv-design
```

**Any agent that loads skills** (Claude Code, Codex, Cursor and others), the skill alone:

```bash
npx skills add getv-design/skills
```

**By hand**: copy `skills/vdesign` to where your agent reads skills, keeping the folder's
name. Claude Code reads `~/.claude/skills/`; many other agents read `~/.agents/skills/`.

**Claude and ChatGPT apps**: add vDesign from the app's connector directory instead. The
connector serves this same skill, so there is nothing to copy.

## What an agent can do with it

**With the connector**, the agent creates and edits documents in your vDesign account, shows
you a live preview in the chat, checks its own layout, and exports PNG, JPEG, WebP, SVG,
PDF, Illustrator, HTML and PowerPoint.

**With the skill alone**, the agent writes a `.vd` file and gives it to you. Open it in the
[vDesign app](https://getv.design/app) or the VS Code extension to see it, keep editing and
export it.

Documentation: [getv.design/docs](https://getv.design/docs).

## This repository is generated

It is published from vDesign's source every time the skill changes there, and each commit
here is one such update. Changes made directly in this repository are overwritten by the next
one, so please don't open pull requests here.

## Licence

The skill and the `.vd` format specification are licensed under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/): use, share and adapt them freely,
crediting "vDesign". See `LICENSE`. The `.vd` format may be implemented by anyone, free of
charge. The vDesign app and connector are separate and are not covered by this licence.
