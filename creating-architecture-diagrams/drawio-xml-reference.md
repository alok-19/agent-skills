# Draw.io XML Reference for Architecture Diagrams

Heavy reference for XML patterns, icon libraries, styles, and templates.
Use alongside SKILL.md when constructing diagrams.

## XML Skeleton

```xml
<mxfile host="app.diagrams.net">
  <diagram id="arch" name="Architecture">
    <mxGraphModel dx="1285" dy="891" grid="1" gridSize="10" guides="1"
                  tooltips="1" connect="1" arrows="1" fold="1" page="0"
                  pageScale="1" pageWidth="1100" pageHeight="900"
                  background="#FFFFFF" math="0" shadow="0">
      <root>
        <mxCell id="0" />
        <mxCell id="1" parent="0" />
        <!-- All nodes, containers, and edges go here -->
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

**Key attributes:**
- `page="0"` — infinite canvas (no page boundaries)
- `background="#FFFFFF"` — white background for professional look
- `grid="1" gridSize="10"` — 10px grid for alignment

## Title Banner

```xml
<mxCell id="title" parent="1" vertex="1"
        style="text;html=1;strokeColor=#FF6D00;fillColor=#FFF3E0;rounded=1;
               align=center;verticalAlign=middle;fontSize=14;fontStyle=1;
               fontColor=#232F3E;"
        value="System Name — Architecture">
  <mxGeometry height="28" width="720" x="140" y="5" as="geometry" />
</mxCell>
```

## AWS Service Icons

All AWS icons use this base pattern:

```xml
<mxCell id="node_id" parent="1" vertex="1"
        style="outlineConnect=0;fontColor=#232F3E;fillColor=<CATEGORY_COLOR>;
               strokeColor=none;verticalLabelPosition=bottom;verticalAlign=top;
               align=center;html=1;fontSize=10;aspect=fixed;
               shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.<ICON>;"
        value="<b>Service Name</b>&#xa;(Description)">
  <mxGeometry height="64" width="64" x="X" y="Y" as="geometry" />
</mxCell>
```

### AWS Icon Color Categories

| Category | fillColor | Services |
|----------|-----------|----------|
| **Compute** | `#D45B07` | ECS, Lambda, EC2, Batch, Fargate |
| **Storage** | `#2E73B8` | S3, EBS, EFS, Glacier |
| **Database** | `#2E73B8` | RDS, DynamoDB, ElastiCache, Redshift |
| **Networking** | `#8C4FFF` | CloudFront, Route53, VPC, ELB, API Gateway* |
| **App Integration** | `#CC2264` | EventBridge, SNS, SQS, Step Functions, API Gateway* |
| **Security** | `#CC2264` | IAM, Cognito, KMS, WAF |
| **Analytics** | `#8C4FFF` | Athena, Glue, Kinesis, QuickSight |

*API Gateway can use either Networking or App Integration color.

### Common AWS Icon Names

| Service | resIcon Value |
|---------|--------------|
| S3 | `mxgraph.aws4.s3` |
| Lambda | `mxgraph.aws4.lambda` |
| API Gateway | `mxgraph.aws4.api_gateway` |
| RDS | `mxgraph.aws4.rds` |
| DynamoDB | `mxgraph.aws4.dynamodb` |
| ElastiCache | `mxgraph.aws4.elasticache` |
| CloudFront | `mxgraph.aws4.cloudfront` |
| ECS | `mxgraph.aws4.ecs` |
| EC2 | `mxgraph.aws4.ec2` |
| EventBridge | `mxgraph.aws4.eventbridge` |
| Step Functions | `mxgraph.aws4.step_functions` |
| SNS | `mxgraph.aws4.sns` |
| SQS | `mxgraph.aws4.sqs` |
| Cognito | `mxgraph.aws4.cognito` |
| VPC | `mxgraph.aws4.vpc` |
| ELB/ALB | `mxgraph.aws4.application_load_balancer` |
| Route 53 | `mxgraph.aws4.route_53` |
| CloudWatch | `mxgraph.aws4.cloudwatch` |
| Kinesis | `mxgraph.aws4.kinesis` |
| EKS | `mxgraph.aws4.eks` |
| Fargate | `mxgraph.aws4.fargate` |
| IAM | `mxgraph.aws4.iam` |
| WAF | `mxgraph.aws4.waf` |
| Secrets Manager | `mxgraph.aws4.secrets_manager` |
| Athena | `mxgraph.aws4.athena` |
| Glue | `mxgraph.aws4.glue` |
| Redshift | `mxgraph.aws4.redshift` |
| SageMaker | `mxgraph.aws4.sagemaker` |

### GCP Icons

GCP icons use `shape=mxgraph.gcp2.<icon>`:

```xml
style="shape=mxgraph.gcp2.cloud_run;fillColor=#4285F4;..."
```

Common: `cloud_run`, `cloud_functions`, `cloud_storage`, `bigquery`,
`cloud_sql`, `pub_sub`, `cloud_cdn`, `compute_engine`, `gke`

### Azure Icons

Azure icons use `shape=mxgraph.azure.<icon>`:

```xml
style="shape=mxgraph.azure.app_service;fillColor=#0078D4;..."
```

Common: `app_service`, `functions`, `blob_storage`, `cosmos_db`,
`sql_database`, `event_grid`, `front_door`, `aks`, `api_management`

## Consumer/User Boxes

For non-service nodes (users, external systems, pipelines):

```xml
<mxCell id="consumer" parent="1" vertex="1"
        style="rounded=1;whiteSpace=wrap;html=1;fillColor=#f5f5f5;
               strokeColor=#888888;fontColor=#111111;fontSize=11;"
        value="<b>Consumer Name</b>">
  <mxGeometry height="40" width="150" x="X" y="Y" as="geometry" />
</mxCell>
```

## Containers (Logical Boundaries)

### Swimlane Container (with title)

```xml
<mxCell id="container" parent="1" vertex="1"
        style="swimlane;startSize=22;fillColor=none;strokeColor=#555;
               dashed=1;html=1;fontStyle=1;fontSize=10;pointerEvents=0;
               collapsible=0;labelBackgroundColor=#ffffff;align=left;spacingLeft=6;"
        value="Layer Name">
  <mxGeometry height="300" width="600" x="X" y="Y" as="geometry" />
</mxCell>
```

### Important Container Rules

1. **Do NOT parent nodes to containers** — keep all nodes at `parent="1"` for simpler edge routing
2. **`pointerEvents=0`** — allows edges to cross container boundaries freely
3. **`fillColor=none`** — transparent so nodes inside are visible
4. **`dashed=1`** — visually distinct from service icons
5. **Container bounds must enclose its logical children** with ≥10px padding

## Edge Patterns

### Base edge (all edges start with this)

```xml
<mxCell id="e1" edge="1" parent="1" source="src" target="tgt"
        style="edgeStyle=orthogonalEdgeStyle;rounded=1;html=1;
               strokeWidth=2;strokeColor=#555;fontColor=#333;fontSize=9;
               labelBackgroundColor=#fff;"
        value="Edge Label">
  <mxGeometry relative="1" as="geometry" />
</mxCell>
```

### Edge with explicit waypoints

```xml
<mxCell id="e2" edge="1" parent="1" source="src" target="tgt"
        style="edgeStyle=orthogonalEdgeStyle;rounded=1;html=1;
               strokeWidth=2;strokeColor=#555;fontColor=#333;fontSize=9;
               labelBackgroundColor=#fff;
               exitX=0.5;exitY=1;entryX=0.5;entryY=0;"
        value="Edge Label">
  <mxGeometry relative="1" as="geometry">
    <Array as="points">
      <mxPoint x="220" y="360" />
      <mxPoint x="480" y="360" />
    </Array>
  </mxGeometry>
</mxCell>
```

### Dashed async edge

Add `dashed=1;` to the style string.

### Highlighted/emphasized edge

```
strokeWidth=4;strokeColor=#0050ef;dashed=1;fontColor=#0050ef;fontStyle=1;
```

### Edge with custom exit/entry points

```
exitX=0.75;exitY=1;   → exits 75% from left on bottom face
entryX=0.5;entryY=0;  → enters center of top face
```

## Legend Box

```xml
<mxCell id="legend" parent="1" vertex="1"
        style="rounded=1;whiteSpace=wrap;html=1;fillColor=#f0f4ff;
               strokeColor=#6c8ebf;fontSize=9;align=left;spacingLeft=8;
               spacingTop=5;verticalAlign=top;fontColor=#232F3E;"
        value="<b><font style='font-size: 10px;'>Legend</font></b>
               <br><font color='#555555'>─── Synchronous call</font>
               <br><font color='#555555'>- - - Async / background</font>
               <br><font color='#0050ef'><b>- - - Special Path</b></font>">
  <mxGeometry height="72" width="148" x="16" y="830" as="geometry" />
</mxCell>
```

## Color Palette

### Professional color palette for architecture diagrams

| Use | Color | Hex |
|-----|-------|-----|
| Title background | Light orange | `#FFF3E0` |
| Title border | Orange | `#FF6D00` |
| Container border | Dark gray | `#555555` |
| Edge stroke | Dark gray | `#555555` |
| Label text | Dark gray | `#333333` |
| Label background | White | `#FFFFFF` |
| Consumer box fill | Light gray | `#f5f5f5` |
| Consumer box border | Medium gray | `#888888` |
| Legend fill | Light blue | `#f0f4ff` |
| Legend border | Medium blue | `#6c8ebf` |
| Highlighted edge | Blue | `#0050ef` |
| Base font color | AWS dark | `#232F3E` |

## Coordinate Cheat Sheet

### Standard icon node
```
Width: 64, Height: 64
Label appears below icon (verticalLabelPosition=bottom)
Label adds ~28px below the icon (2 lines × ~14px)
Total footprint: 64×92 (icon + label)
```

### Standard consumer box
```
Width: 150, Height: 40
Label is inside box (centered)
Total footprint: 150×40
```

### Exit/Entry point pixel positions (64×64 icon)
```
exitX=0   → x = node_x         (left edge)
exitX=0.25→ x = node_x + 16
exitX=0.5 → x = node_x + 32    (center)
exitX=0.75→ x = node_x + 48
exitX=1   → x = node_x + 64    (right edge)

exitY=0   → y = node_y          (top edge)
exitY=0.5 → y = node_y + 32     (center)
exitY=1   → y = node_y + 64     (bottom edge)
```

## Complete Minimal Example

A 3-node diagram with proper grid, exits, corridors, and legend:

```xml
<mxfile host="app.diagrams.net">
  <diagram id="example" name="Minimal Architecture">
    <mxGraphModel dx="800" dy="600" grid="1" gridSize="10" guides="1"
                  tooltips="1" connect="1" arrows="1" fold="1" page="0"
                  pageScale="1" pageWidth="600" pageHeight="400"
                  background="#FFFFFF">
      <root>
        <mxCell id="0" />
        <mxCell id="1" parent="0" />

        <!-- Title -->
        <mxCell id="title" parent="1" vertex="1"
                style="text;html=1;fillColor=#FFF3E0;strokeColor=#FF6D00;
                       rounded=1;align=center;verticalAlign=middle;
                       fontSize=14;fontStyle=1;fontColor=#232F3E;"
                value="Minimal Architecture">
          <mxGeometry height="28" width="400" x="40" y="5" as="geometry" />
        </mxCell>

        <!-- Grid: col=0 x=40, col=1 x=200, col=2 x=360 -->
        <!-- Grid: row=0 y=50, row=1 y=197 -->

        <!-- Nodes -->
        <mxCell id="client" parent="1" vertex="1"
                style="rounded=1;whiteSpace=wrap;html=1;fillColor=#f5f5f5;
                       strokeColor=#888888;fontColor=#111111;fontSize=11;"
                value="&lt;b&gt;Client&lt;/b&gt;">
          <mxGeometry height="40" width="120" x="40" y="50" as="geometry" />
        </mxCell>

        <mxCell id="api" parent="1" vertex="1"
                style="outlineConnect=0;fontColor=#232F3E;fillColor=#CC2264;
                       strokeColor=none;verticalLabelPosition=bottom;
                       verticalAlign=top;align=center;html=1;fontSize=10;
                       aspect=fixed;shape=mxgraph.aws4.resourceIcon;
                       resIcon=mxgraph.aws4.api_gateway;"
                value="&lt;b&gt;API Gateway&lt;/b&gt;">
          <mxGeometry height="64" width="64" x="200" y="165" as="geometry" />
        </mxCell>

        <mxCell id="db" parent="1" vertex="1"
                style="outlineConnect=0;fontColor=#232F3E;fillColor=#2E73B8;
                       strokeColor=none;verticalLabelPosition=bottom;
                       verticalAlign=top;align=center;html=1;fontSize=10;
                       aspect=fixed;shape=mxgraph.aws4.resourceIcon;
                       resIcon=mxgraph.aws4.rds;"
                value="&lt;b&gt;Database&lt;/b&gt;">
          <mxGeometry height="64" width="64" x="360" y="165" as="geometry" />
        </mxCell>

        <!-- Edges -->
        <mxCell id="e1" edge="1" parent="1" source="client" target="api"
                style="edgeStyle=orthogonalEdgeStyle;rounded=1;html=1;
                       strokeWidth=2;strokeColor=#555;fontColor=#333;
                       fontSize=9;labelBackgroundColor=#fff;"
                value="API Request">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>

        <mxCell id="e2" edge="1" parent="1" source="api" target="db"
                style="edgeStyle=orthogonalEdgeStyle;rounded=1;html=1;
                       strokeWidth=2;strokeColor=#555;fontColor=#333;
                       fontSize=9;labelBackgroundColor=#fff;"
                value="Query">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>

        <!-- Legend -->
        <mxCell id="legend" parent="1" vertex="1"
                style="rounded=1;whiteSpace=wrap;html=1;fillColor=#f0f4ff;
                       strokeColor=#6c8ebf;fontSize=9;align=left;
                       spacingLeft=8;spacingTop=5;verticalAlign=top;
                       fontColor=#232F3E;"
                value="&lt;b&gt;Legend&lt;/b&gt;&lt;br&gt;─── Synchronous call">
          <mxGeometry height="36" width="130" x="40" y="310" as="geometry"/>
        </mxCell>
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```
