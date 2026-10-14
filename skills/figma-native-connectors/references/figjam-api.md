# FigJam connector API recipes

Use only in FigJam unless current runtime documentation explicitly supports the operation elsewhere. Follow the loaded `figma-use` execution contract. These snippets are for `use_figma`, not standalone plugins.

## Create an attached, labeled connector

Replace the example node IDs with IDs obtained from the target board. Inspect existing connections first; this creation recipe is not idempotent. Change only the requested styles or match the surrounding flow.

```javascript
if (figma.editorType !== 'figjam') {
  throw new Error('This connector creation recipe requires FigJam.');
}
const source = await figma.getNodeByIdAsync('123:456');
const target = await figma.getNodeByIdAsync('123:789');
if (!source || !target || !('visible' in source) || !('visible' in target)) {
  throw new Error('Resolve both scene-node endpoints before creating a connector.');
}
const pageOf = node => {
  while (node && node.type !== 'PAGE') node = node.parent;
  return node;
};
if (pageOf(source)?.id !== figma.currentPage.id ||
    pageOf(target)?.id !== figma.currentPage.id) {
  throw new Error('Endpoints must be on the target board page.');
}

const font = { family: 'Inter', style: 'Medium' };
await figma.loadFontAsync(font);
const connector = figma.createConnector();
connector.name = 'Review → Done';
connector.connectorLineType = 'ELBOWED';
connector.connectorStart = { endpointNodeId: source.id, magnet: 'RIGHT' };
connector.connectorEnd = { endpointNodeId: target.id, magnet: 'LEFT' };
connector.connectorStartStrokeCap = 'NONE';
connector.connectorEndStrokeCap = 'ARROW_LINES';
connector.text.fontName = font;
connector.text.characters = 'Approved';

return {
  createdNodeIds: [connector.id],
  mutatedNodeIds: [],
  type: connector.type,
  start: connector.connectorStart,
  end: connector.connectorEnd,
  lineType: connector.connectorLineType,
  label: connector.text.characters
};
```

For an unlabeled connector, omit font and text operations. If the chosen font is unavailable, inspect `listAvailableFontsAsync()` and select an available font consistent with the board before creating the node.

## Inspect or repair an existing connector

Resolve the node and check `node.type === 'CONNECTOR'` before accessing connector-specific fields. Return its endpoints, caps, line type, text, and requested style fields. Where available, `attachedConnectors` on an endpoint node helps locate edges; otherwise inspect the board's `CONNECTOR` nodes. Scope searches to the relevant area.

For relinking, replace the complete `connectorStart` or `connectorEnd` value. Validate the new target first. For label edits, load every current font from `connector.text.getRangeAllFontNames(0, connector.text.characters.length)` when the existing label is nonempty. For an empty label, explicitly load and assign a known available font. Preserve existing typography unless changing it is requested.

When converting to `STRAIGHT`, set the line type and reassign both attached endpoints using `CENTER`, or use the supported fixed-position form. When converting to `ELBOWED`, reassign both endpoints with compatible magnets such as `AUTO`, `RIGHT`, or `LEFT`. Verify both final endpoint values.

## Endpoint forms and styling

- Attached and routed: `{ endpointNodeId: id, magnet: 'AUTO' }`, or an allowed side for the chosen line type.
- Attached at a fixed point: `{ endpointNodeId: id, position: { x, y } }`. The position is relative to the target node. Check current coordinate semantics before computing custom offsets; do not treat it as a canvas point or assume normalized 0–1 coordinates.
- Floating: `{ position: { x, y } }`, a point on the canvas. This does not follow a target node.

Set explicit start and end caps when direction matters: `NONE` / `ARROW_LINES` for a one-way flow; arrows at both ends only for a bidirectional relationship. Other supported caps include `TRIANGLE_FILLED`, `DIAMOND_FILLED`, and `CIRCLE_FILLED`; see the [current cap enum](https://developers.figma.com/docs/plugins/api/ConnectorStrokeCap/) for ERD markers.

Use `strokes`, `strokeWeight`, and `dashPattern` for line styling. Reassign the strokes array; RGB channels are 0–1 and opacity belongs on the paint. Connector label text and `textBackground` have separate paints, so changing the line color does not style the label. `cornerRadius` is read-only in the current connector declaration.

Return affected connector IDs plus final properties after every mutation. If a batch fails after partial creation, inspect and finish or remove only its known incomplete nodes rather than running the whole creation batch again.
