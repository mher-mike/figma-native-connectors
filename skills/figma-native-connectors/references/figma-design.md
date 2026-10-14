# Native connectors in Figma Design

Figma documents copy/paste of FigJam objects into Design, where connectors remain special objects. Direct `createConnector()` is documented for FigJam. In a live Design test on 2026-10-14, native paste and UI duplication produced `CONNECTOR` nodes whose `connectorStart` and `connectorEnd` fields could be reassigned through `use_figma`. Creation, API cloning, and other special-object operations are separate capabilities.

## Reuse a native connector

1. Inspect the target Design file for a suitable native connector. Identify it through returned `CONNECTOR` metadata or native connector editing controls. Prefer a connector in the same target area with the desired style.
2. Use native UI duplication or copy/paste to obtain additional connectors. Inspect the pasted/duplicated node's type and new ID. Then use `use_figma` to assign both endpoint objects to the intended node IDs and read the values back. This path was live-tested; fall back to native endpoint handles if the current runtime rejects it. Copied endpoints may still reference their old nodes, or may become fixed positions attached to the enclosing section.
3. If there is no native connector to reuse, copy one from an accessible FigJam board into the target Design file using native copy/paste. If the task needs a new seed, create it in the intended source board with the FigJam API. A screenshot, SVG, or rendered image will not preserve connector behavior.
4. For an interaction flow, attach the start to the specific actionable button/list-item frame, even when nested several levels inside auto layout; attach the end to the intended screen or control. Set labels and styles only through supported properties or visible native controls. Keep the connector outside components/instances and clipped screen content. Parenting it to the surrounding section worked in the live test.
5. Inspect the result and confirm both attachments. If the runtime exposes endpoint metadata, check the destination IDs. Otherwise verify native snapping/attachment controls, with a disposable-copy movement check only if needed. Remove task-created test objects.

Do not use successful endpoint setters as evidence that `createConnector()` or `clone()` also work in Design. Prefer a documented creation API if one becomes available; until then use the native seed workflow. Do not repeatedly probe rejected calls or synthesize undocumented clipboard payloads.

## Test attachment under real movement

Capture the original screen/control positions, sizes, parents, auto-layout modes, sibling order, connector endpoint IDs, and connector bounds. Move the source's containing screen and the destination screen independently. Do not reassign endpoints or manually move connectors during the test. Read back stable endpoint IDs and changed geometry, then inspect the canvas for correct routing.

When the user requests a control-level test, also exercise layout-driven movement. For example, temporarily reorder the connected button or list item inside its existing auto-layout parent, observe the connector following the same control, and restore its original sibling order. Load only the moved controls' actual fonts before reparenting/reordering; avoid a broad title-font preload or substituting unavailable fonts elsewhere in the screen.

Restore original positions, sizes, parent relationships, and sibling order, then verify them against the baseline. Keep the final useful connectors and remove task-created seed/test leftovers. An error may roll back mutations: inspect actual state before assuming either partial changes or successful restoration.

The 2026-10-14 test passed for a button nested in a content frame and a list item nested in a separate screen's content frame. Both connectors retained their source IDs and rerouted after all parent-screen moves and after control reordering, without any endpoint assignments during those movements. This is verified evidence for that workflow, not a guarantee for every nesting or editor version.

## When a transfer needs user help

Use the available native UI tools when they can complete the authorized edit. If the session cannot access the source board, destination file, or native clipboard workflow, identify the exact missing step. For example: “Paste one FigJam connector into this Design file, then share its node link so I can continue.” Ask only for what is actually missing; do not move the task into FigJam or replace connectors with vector arrows without the user's agreement.

Do not create an unrelated board just to obtain a connector. If a new scratch board is necessary, establish that destination with the user and load `figma-create-new-file` before creating it. Keep API, native UI, and manual steps distinct when describing what was completed.

## Primary source

[Work between Figma and FigJam](https://help.figma.com/hc/en-us/articles/1500005916661-Work-between-Figma-and-FigJam) documents the special-object workflow and its limitations. Checked 2026-10-14, supplemented by the live test above. Verify native UI behavior and API mutation results in the actual target file.
