---
name: figma-native-connectors
description: Create, edit, and repair native smart flow connectors in Figma Design and FigJam, including attached endpoints, routing, arrows, and labels. Use for wiring screens or diagram nodes with connectors that follow their targets. Excludes prototype interactions, Code Connect, and decorative vector arrows.
---

# Figma Native Connectors

Work with editable native connectors whose endpoints stay attached to their intended nodes. Preserve the user's chosen file and editor.

## Choose the supported route

Load the installed `figma:figma-use` skill before using `use_figma`. For FigJam API work, also load `figma:figma-use-figjam`. Resolve these skills through the current skill catalog; do not hardcode a plugin cache version.

| Destination | Route |
| --- | --- |
| FigJam (`/board/`, `editorType: "figjam"`) | Use `figma.createConnector()` and the connector API. Read [FigJam API recipes](references/figjam-api.md). |
| Figma Design (`/design/`, `editorType: "figma"`) | Seed or duplicate native connectors through the observed native UI. Existing connector endpoint setters were live-tested successfully through `use_figma`; inspect and verify each mutation. Read [Design workflow](references/figma-design.md). |

The public API currently documents `createConnector()` as FigJam-only, and the installed MCP guide blocks connector creation in Design. This differs from editing an existing pasted connector: `connectorStart` and `connectorEnd` setters worked in the live Design test on 2026-10-14. Do not extrapolate that result to direct creation, API cloning, or unrelated special-object operations. Check each operation's result and stop repeating a rejected mutation.

Native connectors, vector arrows, and prototype interactions serve different purposes. Do not silently substitute one for another. Creating this skill or discussing connectors does not itself call for changing a design file.

## Inspect and plan the connections

1. Resolve the destination file and target nodes from the user's link/context. If the destination is missing, ask for its link. Use `get_figjam` for FigJam inspection and `get_metadata` or a narrow read-only `use_figma` for Design. Do not use `get_metadata` on FigJam.
2. Identify each intended source, destination, direction, and optional label. Use discovered IDs, not guessed IDs or stale selections. Check the target page and existing connectors before adding more.
3. Inspect existing style and routing. For a repair, change the existing connector where supported. Do not recreate a whole flow to fix one edge. Two nodes may legitimately have several differently labeled relationships.

For interactive user flows, attach the source to the actual button, list item, link, or other action control, including when it is nested inside auto layout. A screen-to-screen overview does not test control-level attachments. Use screen-level sources when the user asks for an overview; otherwise resolve the triggering control and the intended destination.

## Preserve native behavior

- Verify the object is a native `CONNECTOR` through API metadata or native connector controls. An arrow-shaped vector is not sufficient evidence.
- Attach to the actual intended frame, shape, sticky, or other target. A free endpoint placed against its edge only looks attached. Use floating endpoints only when requested.
- Prefer elbowed routing with `AUTO` or explicit edge magnets for ordinary screen flows. **Straight connectors accept `CENTER` or `NONE` magnets; elbowed connectors do not accept `CENTER`.** Configure the line type before assigning compatible endpoints. For curved routing, inspect existing behavior or current documentation rather than extrapolating straight-line rules.
- Respect the existing layout. Use sensible edge choices and visible gaps to avoid crossing screen contents or covering labels. Do not rearrange the user's screens unless the task includes layout changes.
- Native label text belongs to `connector.text.characters`; `connector.name` only names the layer. Load fonts before text mutations. A new connector needs an explicitly loaded and assigned font; its initial text font may be invalid.
- Keep the connector in a context where it remains visible and editable. In Design, connectors cannot be packaged inside components or instances. Do not detach components to make room for them.

## Execute and verify

For `use_figma`, write plain JavaScript with top-level `await` and `return`. Do not copy standalone-plugin wrappers or `figma.closePlugin()` calls from older connector examples. Include the applicable installed Figma skill names in `skillNames` as required by those skills. Return all created/mutated node IDs, endpoint data, line types, and labels.

Check attachment IDs, direction, and label text from the result. Inspect a screenshot or the native canvas for routing, readability, and clipping. For a UI-only attachment, confirm snapping/attachment through native controls; if still uncertain, test movement on a disposable copy and remove the copy afterward. Never report attachment as verified from visual proximity alone.

After an error or uncertain completion, obey the tool's retry guidance and inspect the affected area when needed. Reconcile partial work before retrying so a timeout does not create duplicate edges. If the chosen route is unavailable, state the exact missing capability or user action; keep the intended destination and native-connector requirement intact.

Report what changed, link the destination, and distinguish structural/visual checks from any untested movement behavior.

## Sources and maintenance

Checked 2026-10-14 against the installed Figma plugin and these primary sources:

- [Connector API and creation](https://developers.figma.com/docs/plugins/api/figma/#createconnector)
- [Connector node properties](https://developers.figma.com/docs/plugins/api/ConnectorNode/)
- [Endpoint compatibility](https://developers.figma.com/docs/plugins/api/ConnectorEndpoint/)
- [Figma–FigJam copy/paste and special-object limits](https://help.figma.com/hc/en-us/articles/1500005916661-Work-between-Figma-and-FigJam)

Recheck current docs when editor support differs from this snapshot. Live Design testing on 2026-10-14 verified native copy/paste, UI duplication, endpoint relinking to nested button/list-item frames, automatic rerouting after parent-screen movement and control reordering, and restoration of the original layout. This establishes those operations in the tested runtime; validate the actual target file rather than assuming every special-object operation works.
