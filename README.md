# Figma Native Connectors

A Codex skill for native flow connectors in **Figma Design and FigJam**. Connect actual buttons and list items to their destinations, with endpoints that follow their targets when the layout moves.

[Download the latest ZIP](https://github.com/mher-mike/figma-native-connectors/releases/latest) · [Read the skill](skills/figma-native-connectors/SKILL.md)

## What it does

- Attaches native connectors to buttons, list items, frames, and diagram nodes.
- Guides routing, arrowheads, labels, and repairs to existing connectors.
- Verifies attachment using node IDs and movement, including controls inside auto layout.
- Preserves the original layout after a movement test.

This is an **agent skill**, not a Figma plugin or a prototype interaction generator.

## Requirements

- Codex with local skill support.
- The official Figma plugin, connected to a Figma account with edit access. The workflow expects the `use_figma` write tool and the plugin's `figma-use` and `figma-use-figjam` skills. A read-only Figma MCP connection alone is insufficient.
- For Design: access to native Figma copy/paste and duplication through computer-use tools, or a manual paste of a native connector when prompted.

The Design workflow reuses native connectors or transfers one from FigJam. Direct connector creation through `figma.createConnector()` is documented for FigJam. Editing the endpoints of an existing Design connector was verified in the live test below. Runtime support can vary; the skill checks the actual result.

## Install

Paste this into Codex:

```text
$skill-installer Install the skill at https://github.com/mher-mike/figma-native-connectors/tree/main/skills/figma-native-connectors into my personal skills folder. Then check whether the required Figma write tool and companion skills are available.
```

Or download `figma-native-connectors-v1.0.0.zip` from [Releases](https://github.com/mher-mike/figma-native-connectors/releases), extract it, and ask Codex:

```text
Install the figma-native-connectors skill from this extracted folder into my personal skills folder. Preserve SKILL.md, agents/, and references/. Then check the Figma tool requirements in README.md.
```

Use the skill installer for the correct location in your setup. Current public documentation lists `~/.agents/skills` for user skills; some Codex installations use `~/.codex/skills`. Install in one supported location. If the skill does not appear, restart Codex. See [OpenAI's skill documentation](https://learn.chatgpt.com/docs/build-skills).

## Use

```text
Use $figma-native-connectors in this Figma file: [your Figma URL].
Connect the Continue button to the next screen and the first list item to its detail screen. Use native connectors attached to those controls. Test that they follow when the parent frames move and when the controls reorder in auto layout, then restore the original layout.
```

For FigJam:

```text
Use $figma-native-connectors in this FigJam board: [your FigJam URL].
Connect Review to Approved with an elbowed native connector, a one-way arrow, and the label "Approved". Verify both endpoint attachments.
```

## What was tested

Live-tested in Figma Design through Codex desktop on macOS on **2026-10-14**:

| Check | Result |
| --- | --- |
| Native connector copied from FigJam into Design | Passed |
| Native connector duplicated through the Design UI | Passed |
| Start endpoints attached to a nested button and list item | Passed |
| Source and destination screens moved independently | Attachments retained and geometry followed |
| Button and list item reordered within auto layout | Attachments retained and geometry followed |
| Original positions, dimensions, parents, and sibling orders restored | Passed |

During movement tests, endpoint IDs were unchanged and endpoints were not reassigned. The skill includes FigJam API recipes; this release does not claim that every recipe, nesting arrangement, or editor version has been live-tested. Direct Design creation and API cloning were not established by these tests.

## Files

```text
skills/figma-native-connectors/
├── SKILL.md
├── agents/openai.yaml
└── references/
    ├── figma-design.md
    └── figjam-api.md
```

The release ZIP contains the installable `figma-native-connectors/` folder, this README, and the license. No account configuration or sample design file is required.

## License

[MIT](LICENSE). Free to use, modify, and share. This is a community skill, unaffiliated with Figma or OpenAI.
