# skills

Skills for Claude Code and Codex, distributed as a marketplace. Each plugin groups the skills for one
platform, so installing one does not put unrelated skills in front of the model.

## Layout

Every skill lives inside the plugin that carries it, as ordinary files:

```
plugins/uno-skills/.claude-plugin/plugin.json      the plugin manifest
plugins/uno-skills/skills/uno-ux-design/           the skill itself
plugins/uno-skills/skills/uno-design-review/       ... and the review it feeds
```

A plugin's `skills/` therefore reads as the list of what it carries, which is what one plugin per
platform is meant to give you.

There is no shared `skills/` tree at the root with symlinks pointing into it. `codex plugin add`
copies a plugin into `~/.codex/plugins/cache/` and does **not** follow symlinks - a linked skill
installs as an empty directory and Codex reports no error, so the plugin looks installed and carries
nothing. Plain files are the only layout both clients read the same way, and they also make
`claude plugin validate plugins/uno-skills` pass, which a symlinked tree does not.

Both clients read the same `.claude-plugin/` manifests; Codex accepts them alongside its own
`.codex-plugin/` format, so nothing here is duplicated per client.

## Install

Claude Code:

```
/plugin marketplace add Krzysztof318/skills
/plugin install uno-skills@krzysztof318-skills
```

Codex:

```bash
codex plugin marketplace add Krzysztof318/skills
codex plugin add uno-skills@krzysztof318-skills
```

For local work, point either client at the checkout directly:

```bash
claude --plugin-dir /path/to/skills/plugins/uno-skills
codex plugin marketplace add /path/to/skills   # then codex plugin add, as above
```

Codex installs a snapshot rather than reading the checkout live: after changing a skill locally, run
`codex plugin marketplace upgrade` (Git sources) or re-add the local marketplace, then reinstall the
plugin. Claude Code reads `--plugin-dir` from disk on every start.

## Plugins

| Plugin | Skills | For |
| --- | --- | --- |
| `uno-skills` | `uno-ux-design`, `uno-design-review` | Uno Platform application UI |

### `uno-skills`

Uno documents its own surface, and Uno Platform Studio ships skills covering MVUX, navigation, the
Toolkit, and theming. This plugin does not restate any of it. It carries what those leave out: the
judgement about how a screen should be designed, and whether it is good enough to ship.

- **`uno-ux-design`** - run before the XAML exists. The first half of a screen: the one action,
  the hierarchy, the density, the states, the motion, and the few style decisions worth making,
  with the numbers an Uno screen starts from - the spacing scale, the Toolkit's breakpoints, the
  touch-target minimums, the motion durations. Its handoff is the review.
- **`uno-design-review`** - run before shipping a screen. A ledger of the defects that survive a
  green build, each with the signature that reveals it, the search that finds it, and the response
  that repairs it. Covers hierarchy, theming discipline, adaptive layout, list virtualization,
  state completeness, form usability, navigation integrity, platform feel, visual consistency,
  accessibility, and placeholder content. Uses the Uno App MCP for the visual checks where it is
  available, and says so when it is not.

In Claude Code they are invoked as `/uno-skills:uno-ux-design` and
`/uno-skills:uno-design-review`. In Codex they are loaded from the installed plugin and picked up
from their descriptions - ask for a screen design before building, and a screen review before
shipping.
