---
name: creating-architecture-diagrams
description: Use when creating professional draw.io architecture diagrams, system design visualizations, or infrastructure diagrams (AWS, GCP, Azure) that need clean routing with no line overlaps, label collisions, or visual clutter
---

# Creating Architecture Diagrams

## Overview

Professional architecture diagrams require **mathematical precision before visual polish**. Place nodes on a grid, allocate unique exit points per edge, route through dedicated corridors, and verify collisions geometrically — BEFORE opening the diagram visually.

**Core principle:** If you can't prove with coordinates that no lines overlap, they will overlap.

## When to Use

- Creating system architecture or infrastructure diagrams in draw.io XML
- Building AWS/GCP/Azure architecture visualizations
- Designing interview-ready system design diagrams
- Any draw.io diagram where edge routing quality matters

**When NOT to use:**
- Simple flowcharts with <5 nodes (standard draw.io handles these)
- Mermaid diagrams (Mermaid has its own layout engine)
- Diagrams where approximate layout is acceptable

## The 7 Rules

| # | Rule | What It Prevents |
|---|------|-----------------|
| 1 | **Grid-First Placement** — Assign (col, row) to every node before writing XML | Uneven spacing, ad-hoc positions |
| 2 | **Unique Exit Points** — No two edges share the same exitX/exitY on a node | Overlapping line segments at source |
| 3 | **Dedicated Routing Corridors** — Define y-corridors between node rows for horizontal edge runs | Edges crossing through node zones |
| 4 | **Minimum 20px Clearance** — Every edge segment ≥20px from any node boundary | Edges grazing node icons at zoom |
| 5 | **Every Edge Labeled** — Every edge has `value="Label"` with `labelBackgroundColor=#fff` | Unlabeled arrows, guesswork |
| 6 | **Collision Verification** — Mathematically check every edge×node intersection | Hidden overlaps found only after rendering |
| 7 | **Legend Isolation** — Legend placed below all routing buses, verified clear | Legend traversed by edge paths |

## The 8-Step Process

### Step 1: INVENTORY

List all components from requirements:
- **Nodes**: services, databases, queues, users/consumers
- **Edges**: data flows, API calls, events — each with a label
- **Containers**: logical groupings (VPC, subnet, layer, zone)
- **Special**: legend, title, annotations

### Step 2: GRID

Assign each node a (col, row) position using fixed spacing:

```
Node x = col_index × COL_SPACING + LEFT_MARGIN
Node y = row_index × ROW_SPACING + TOP_MARGIN

Recommended defaults:
  COL_SPACING = 160    ROW_SPACING = 147
  LEFT_MARGIN = 40     TOP_MARGIN  = 38
  NODE_SIZE   = 64×64  (for service icons)
  BOX_SIZE    = 150×40 (for consumer/user boxes)
```

**Document the grid as a table before writing XML:**

```
| Node           | Col | Row | x   | y   |
|----------------|-----|-----|-----|-----|
| Web Platform   | 1   | 0   | 200 | 38  |
| API Gateway    | 2   | 1   | 360 | 185 |
| Lambda Search  | 2   | 2   | 360 | 332 |
```

### Step 3: CONTAINERS

Define logical boundaries using a **plain rectangle container** (no swimlane) + a **separate floating text label** pinned just above the top-left corner:

```xml
<!-- Container: plain dashed rectangle, NO label value -->
<mxCell id="c_api" parent="1" vertex="1"
        style="rounded=1;fillColor=none;strokeColor=#555;
               dashed=1;html=1;pointerEvents=0;"
        value="">
  <mxGeometry x="180" y="180" width="104" height="263" as="geometry" />
</mxCell>

<!-- Label: floating text pinned above container top-left corner -->
<mxCell id="lbl_api" parent="1" vertex="1"
        style="text;html=1;strokeColor=none;fillColor=#ffffff;
               align=left;verticalAlign=middle;fontStyle=1;
               fontSize=10;fontColor=#444;"
        value="API Layer">
  <mxGeometry x="184" y="169" width="72" height="16" as="geometry" />
</mxCell>
```

**Label positioning formula:**
```
label_x = container_x + 4          # 4px indent from left border
label_y = container_y - 11         # sits ABOVE the border line (negative offset)
label_w = ~7px × char_count        # e.g. "API Layer" (9 chars) ≈ 66px
label_h = 16
fillColor=#ffffff                   # white pill visually clips through the dashed border
```

**Why above top-left?** The white background creates a "notch" effect — the label appears printed on the border. This is the professional pattern used in cloud architecture diagrams. Arrows that pass nearby still read correctly because the label is a small, left-aligned text before the edge entry zone.

> **Centered container labels** (preferred when container is full-width): set `x = container_x + (container_w / 2) - (label_w / 2)` and use `align=center`. Verify that the centered label's x-span does not overlap any edge entry points on the left or right sides.

### Step 4: EXIT MAP

For each node with multiple outgoing edges, allocate **unique** exit points:

```
Available positions per face:
  Bottom: exitY=1, exitX ∈ {0, 0.25, 0.5, 0.75, 1}
  Right:  exitX=1, exitY ∈ {0, 0.25, 0.5, 0.75, 1}
  Left:   exitX=0, exitY ∈ {0, 0.25, 0.5, 0.75, 1}
  Top:    exitY=0, exitX ∈ {0, 0.25, 0.5, 0.75, 1}
```

**Document the exit map before writing edges:**

```
| Source Node  | Target         | exitX | exitY | entryX | entryY |
|-------------|----------------|-------|-------|--------|--------|
| API Gateway | λ Download     | 0     | 1     | 0.5    | 0      |
| API Gateway | λ Search       | 0.25  | 1     | 0.5    | 0      |
| API Gateway | λ Insights     | 0.75  | 1     | 0.5    | 0      |
```

**NEVER assign the same (exitX, exitY) to two edges on the same source node.**

**If a node has only ONE outgoing edge, always use `exitX=0.5;exitY=1` (bottom center) — do NOT use corner exits.**
Corner exits (exitX=0 or exitX=1 with exitY=1) make lines start from the edge of a box, not its center, which looks unprofessional.

### Step 5: ROUTE

Route each edge with explicit waypoints through corridors:

```
Define corridors FIRST:
  corridor_y_1 = (row_0_bottom + row_1_top) / 2   # between consumer and access rows
  corridor_y_2 = (row_1_bottom + row_2_top) / 2   # between access and lambda rows
  gutter_x     = rightmost_col_x + COL_SPACING     # vertical corridor for special paths
```

Use explicit `<mxPoint>` waypoints for any non-straight path:

```xml
<mxCell id="e1" edge="1" source="api" target="lambda" style="...">
  <mxGeometry relative="1" as="geometry">
    <Array as="points">
      <mxPoint x="448" y="360" />
      <mxPoint x="220" y="360" />
    </Array>
  </mxGeometry>
</mxCell>
```

**Never rely on auto-routing for complex diagrams.** Explicit waypoints = reproducible results.

### Step 6: LABEL

Every edge MUST have a label:

```xml
value="Label Text"
style="...fontColor=#333;fontSize=9;labelBackgroundColor=#fff;..."
```

Rules:
- **Mandatory White Backgrounds**: Always use `labelBackgroundColor=#ffffff` for edge labels. This creates a clean graphical "cutout" effect automatically when lines pass over gridlines or dashed container borders.
- **Label Staggering on Parallel Paths**: If multiple edges run between the same two nodes (e.g. an up-wire and a down-wire), you MUST configure `<mxGeometry x="-0.3" relative="1" ...>` or similar offset to slide their respective labels up/down the string so the texts do not clump and collide.
- **Label Offset on Intersections**: By default, labels sit at the exact median of total edge length (`x="0"`). If an edge perpendicularly crosses another edge at its halfway point, the label will render directly on top of the crossing wire. Use `<mxGeometry x="-0.3" ...>` to slide the label off the collision point.
- No duplicate labels — if two edges serve similar purposes, differentiate ("Auth Cache" vs "Cache Lookup")
- Dashed async edges: add `dashed=1` to the style
- Special emphasized paths: use `strokeWidth=4;strokeColor=#0050ef;fontColor=#0050ef;fontStyle=1`

### Step 7: TITLE & LEGEND

#### Main Title
Never use a plain `text` shape for the main diagram title. Always place the title in a distinctly styled, rounded bounding box at the top center of the canvas to give it a professional presentation:

```xml
<mxCell id="title_box" value="Project Name — System Architecture" 
        style="rounded=1;whiteSpace=wrap;html=1;fillColor=#FFF3E0;strokeColor=#FF6D00;fontColor=#232F3E;fontStyle=1;fontSize=16;align=center;verticalAlign=middle;" 
        vertex="1" parent="1">
  <mxGeometry x="[centered_x]" y="20" width="600" height="40" as="geometry" />
</mxCell>
```

#### Legend
Place the legend below ALL routing corridors:

```text
legend_y = max(all_bus_corridor_y) + 50px
legend_x = LEFT_MARGIN (left side, out of main flow)
```

**CRITICAL: Never construct legend lines using HTML `border-bottom` on `<span>` tags with `&nbsp;`.** Font rendering discrepancies will cause the lines to detach or misalign from the text. Instead, build the legend using genuine geometric `mxCell` layout:

1. One background container (`rounded=1`)
2. One title text box (`<b>Legend</b>`)
3. Separate `<mxPoint>` vector lines mirroring diagram edge styles (e.g. `edgeStyle=none;endArrow=none;strokeWidth=2`)
4. Separate text boxes bounding the actual labels

**Verify**: legend bounding box `(lx, ly, lw, lh)` must not intersect ANY edge path segment.

### Step 8: VERIFY (Three-Phase)

#### Phase 1 — Geometric Audit (before opening draw.io)

Run this check for EVERY edge:

```
For each edge E with segments [(x1,y1)→(x2,y2), ...]:

  COLLISION CHECK:
    For each node N at (nx, ny) with size (nw, nh):
      expanded = (nx-20, ny-20, nw+40, nh+40)  # 20px margin
      IF segment intersects expanded box → FIX routing

  EXIT OVERLAP CHECK:
    For each other edge E2 on same source node:
      IF E.exitX == E2.exitX AND E.exitY == E2.exitY → REALLOCATE

  LABEL CHECK:
    IF E has no value="" attribute → ADD label

  CLEARANCE CHECK:
    For each waypoint (wx, wy):
      For each node N:
        IF |wx - N.center_x| < (N.width/2 + 20) AND
           |wy - N.center_y| < (N.height/2 + 20) → REROUTE
```

Present results as an audit table:

```
| Edge | Route | Collisions | Label | Exit Unique |
|------|-------|------------|-------|-------------|
| e1   | A→B   | ✅ clear   | ✅    | ✅          |
| e2   | B→C   | ⚠️ 12px   | ✅    | ✅          |
```

#### Phase 2 — XML Compilation Check

**CRITICAL: Always validate the XML structure before delivering the diagram.**
Malformed XML (missing closing tags, unescaped quotes) will cause draw.io to throw `Start tag expected` or `Invalid input` errors upon opening.

Run the following terminal command to compile-check the `.drawio` file:
```bash
xmllint --noout /path/to/diagram.drawio
```
If the command outputs ANY errors, you MUST fix the XML syntax before proceeding. Do NOT ask the user to view a diagram that fails this check.

#### Phase 3 — Visual Inspection (ONE pass)

**CRITICAL: Always open via the correct URL to load AWS shape libraries:**
```
https://app.diagrams.net/?libs=aws4
```
Then use `File → Open from Device` or `Extras → Edit Diagram` to load the XML.

**Why this matters:** Opening a `.drawio` file by double-click or file association does NOT auto-load external shape libraries. Without `?libs=aws4`, all `mxgraph.aws4` shapes silently fall back to plain colored rectangles — same color, but no icon graphics. The XML is correct; only the library load is missing.

Alternatively, use the `open_drawio_xml` MCP tool which loads libraries automatically.

Check:
- AWS icons render with actual graphics (door, λ, lightning bolt, bucket) — not plain squares
- Labels appear below icons (`verticalLabelPosition=bottom` working)
- Container floating labels show white notch clipping the dashed border
- Fonts readable at default zoom
- No auto-routing overrides distorting explicit waypoints

**If Phase 2 finds >1 issue, the geometric audit was incomplete — improve it.**

## Quick Reference: Edge Styles

| Type | Style Fragment |
|------|---------------|
| Solid sync | `strokeWidth=2;strokeColor=#555;` |
| Dashed async | `strokeWidth=2;strokeColor=#555;dashed=1;` |
| Highlighted | `strokeWidth=4;strokeColor=#0050ef;dashed=1;fontColor=#0050ef;fontStyle=1;` |
| All edges | `edgeStyle=orthogonalEdgeStyle;rounded=1;html=1;` |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Two edges exit same point on a node | Allocate unique exitX/exitY per edge |
| Edge routes in C-shape loop around target | Change exit face (exitX=0.75 not exitX=1) |
| Legend sits in bus corridor | Move legend below max bus y + 50px |
| Back-and-forth jog in edge path | Remove intermediate waypoint causing reversal |
| Same label on two edges | Use distinct labels per edge |
| Title overlapped by bridge routes | Route bridges in corridor BELOW consumer row |
| Edge grazes node at small zoom | Enforce ≥20px clearance mathematically |
| Relying on auto-routing | Use explicit waypoints for all non-trivial paths |
| Container title shows "=\|" prefix | Add `collapsible=0;` to swimlane container style |
| Container title shows inside box, collides with edges | Use plain rectangle container (value="") + separate floating text label |
| Title bar text appears left-aligned despite `align=center` | Do NOT use `text;` as base style for title bars — it ignores alignment. Use `rounded=1;whiteSpace=wrap;html=1;` as the base style |
| Container floating label intersecting with central edge | Never blindly center container labels. If an edge crosses the boundary top-center or bottom-center, shift the container text box's `x` completely to the left or right, and use `align=left` or `align=right` respectively. |
| Two labels on parallel edges overlapping violently | Shift `entryX` and `exitX` to pry the wires apart horizontally, AND set `x="-0.3"` inside the edge's `mxGeometry` to stagger the labels vertically along the wire. |
| Edge label overlaps a perpendicular intersecting wire | The crossing happened exactly at the 50% distance mark (`x="0"`). Shift the label using `<mxGeometry x="-0.3" relative="1"...>` to push it away from the center intercept. |
| Legend lines misaligned with text at different zoom levels | Stop using HTML `border-bottom` spans. Rebuild the legend using pure geometric `edge` cells and independent `text` cells inside a bounding box. |
| Single-edge node exits from corner not center | Use `exitX=0.5;exitY=1` for nodes with only one outgoing edge |
| AWS icons show as plain colored squares (no graphics) | Open draw.io via `https://app.diagrams.net/?libs=aws4` — file-association open does NOT load shape libraries |
| `mxgraph.aws4.resourceIcon` stencil not rendering via Edit Diagram paste | Use `shape=cylinder3` for databases instead — it's built-in and always renders. Key: omit `whiteSpace=wrap` so `verticalLabelPosition=bottom` works — with wrap, label is forced inside the cylinder |

## Red Flags — STOP and Re-Verify

- Any edge with no `value=""` attribute
- Any two edges with identical exit coordinates on the same node
- Legend bounding box not verified against all edge paths
- Waypoints not explicit (relying on auto-routing for complex paths)
- Corridor y-values not documented before routing
- Grid positions assigned ad-hoc instead of formulaically

**If you see any of these, go back to the relevant step and fix before proceeding.**

See `drawio-xml-reference.md` for complete XML patterns, icon libraries, and style strings.
