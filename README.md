# skills

Skills for Claude Code, distributed as a marketplace. Each plugin groups the skills for one
platform, so installing one does not put unrelated skills in front of the model.

## Layout

Every skill lives once, at the root, under `skills/<name>/`. A plugin does not hold copies: it links
the skills it carries, one symlink per skill.

```
skills/uno-design-review/SKILL.md                     the skill itself
plugins/uno-skills/skills/uno-design-review  ->  ../../../skills/uno-design-review
```

The link is per skill rather than over the whole `skills/` directory on purpose. A directory-wide
link would hand every future skill to whichever plugin holds it, which is exactly what one plugin per
platform is meant to prevent. A plugin's `skills/` therefore reads as the list of what it carries.

Claude Code follows these links when it loads a plugin, both from `--plugin-dir` and from a
marketplace install, where a link resolving elsewhere in the same marketplace is dereferenced into
the plugin cache. `claude plugin validate` does *not* follow them and says so; validate `skills/`
directly, which is the real path.

Git stores the links as symlinks (mode `120000`). A clone on Windows needs `core.symlinks` for them
to arrive as links rather than as text files.

## Install

```
/plugin marketplace add Krzysztof318/skills
/plugin install uno-skills@krzysztof318-skills
```

## Plugins

| Plugin | Skills | For |
| --- | --- | --- |
| `uno-skills` | `uno-design-review` | Uno Platform application UI |

### `uno-skills`

Uno documents its own surface, and Uno Platform Studio ships skills covering MVUX, navigation, the
Toolkit, and theming. This plugin does not restate any of it. It carries what those leave out: the
judgement about whether a screen is good enough to ship.

- **`uno-design-review`** - run before shipping a screen. A ledger of the defects that survive a
  green build, each with the signature that reveals it, the search that finds it, and the response
  that repairs it. Covers theming discipline, adaptive layout, list virtualization, state
  completeness, platform feel, accessibility, and placeholder content. Uses the Uno App MCP for the
  visual checks where it is available, and says so when it is not.

Invoked as `/uno-skills:uno-design-review`.
