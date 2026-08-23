# skills

Skills for Claude Code, distributed as a marketplace. Each plugin groups the skills for one
platform, so installing one does not put unrelated skills in front of the model.

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
