# Fibre-Optic Distribution Network Analysis
## Reusel, Netherlands — FTTH Expansion Viability Assessment

---

## Project Overview

This project assesses the viability of a Fibre-to-the-Home (FTTH) expansion area in **Reusel, Netherlands**, using OpenStreetMap road data as the basis for network routing. The objective is to understand how residential demand connects back to distribution points, and how that demand translates into cable load across the trench network.

---

## Coordinate Reference System

All spatial work is carried out in **ETRS89 / UTM Zone 31N (EPSG:25831)**. All layers are reprojected to this CRS so distances and routing are calculated in metres.

---

## Source Data

| Layer         | Source          | Features | Description                                    |
|---------------|-----------------|----------|------------------------------------------------|
| OSM_Buildings | OpenStreetMap   | 561      | All building footprints in the study area      |
| OSM_Roads     | OpenStreetMap   | 43       | Road network used as routing surface and trench|

---

## Processing Workflow

### 1. Residential Demand Filtering
The raw OSM building dataset (561 features) is filtered to retain only residential demand.
Buildings tagged as `garage` or `school` are excluded. Retained values: `house`, `apartments`, `yes`.

**Result:** 552 residential buildings.

### 2. Building Centroids
Polygon footprints converted to point centroids — demand nodes for network connection.

### 3. Distribution Points
Three DPs placed across the study area extent:

| ID | Name  | Position                     |
|----|-------|------------------------------|
| 1  | DP-01 | West  (20% of E–W extent)   |
| 2  | DP-02 | Centre                       |
| 3  | DP-03 | East  (80% of E–W extent)   |

### 4. Hub Lines
Straight-line connections from each building centroid to its nearest DP (by Euclidean distance).
Used to: visualise demand allocation and intersect with the road network for trench split points.
**Result:** 552 hub lines.

### 5. Hub–Road Intersections
Intersections between hub lines and road network captured as points.
**Result:** 843 intersection points.

### 6. Trench Network Construction
Road layer treated as the physical trench network, fully segmented at:
- All existing road–road intersections
- Every hub line intersection point

Hub lines similarly split at road crossings. Both merged into a **combined trench network** and
subjected to a final self-split to ensure complete segmentation at every node.

| Stage                         | Segments |
|-------------------------------|----------|
| Roads after hub split         | 886      |
| Roads after self-split        | 939      |
| Hub lines after road split    | 1,363    |
| Merged trench                 | 2,302    |
| Final fully-segmented trench  | 2,334    |

### 7. Cable Load (cable_count)
Each trench segment assigned a `cable_count` — number of building connections (hub lines) that
spatially overlap it. Computed via spatial intersection across all hub lines.

| Metric                  | Value |
|-------------------------|-------|
| Total segments          | 2,334 |
| Segments with load > 0  | 1,767 |
| Maximum cable count     | 326   |
| Load classes            | 5     |

### 8. Shortest-Path Routing
Road-network shortest-path routes computed from each DP to all 552 building centroids
using Dijkstra's algorithm (QGIS native:shortestpathpointtolayer).
**Result:** 1,656 routed cable paths (3 DPs × 552 buildings).

### 9. Vertex Extraction
Trench vertices extracted after all intersections and splits are resolved.
**Result:** 4,856 vertices.

---

## Output Layers

| Layer                      | Type    | Description                                             |
|----------------------------|---------|---------------------------------------------------------|
| Buildings_Residential      | Polygon | Reprojected residential buildings (EPSG:25831)          |
| Roads_31831                | Line    | Reprojected road network (EPSG:25831)                   |
| Building_Centroids         | Point   | Residential demand nodes                                |
| Distribution_Points        | Point   | 3 fibre distribution points                             |
| Hub_Lines                  | Line    | Straight-line DP-to-centroid connections                |
| Hub_Road_Intersections     | Point   | Intersections of hub lines with roads                   |
| Roads_Segmented            | Line    | Road network split at all intersections                 |
| Trench_Network_Segmented   | Line    | Combined trench with cable_count (main deliverable)     |
| Trench_Vertices            | Point   | All trench network vertices                             |
| Routed_Cables              | Line    | Shortest-path routes from DPs to buildings              |

---

## Symbology

### Trench Network — Graduated by cable_count

| Class     | Range   | Colour           | Width   |
|-----------|---------|------------------|---------|
| No load   | 0       | #d3d3d3 (grey)   | 0.35 px |
| Low       | 1–10    | #fee391 (yellow) | 0.55 px |
| Medium    | 11–50   | #fe9929 (orange) | 0.85 px |
| High      | 51–150  | #d94801 (red-or) | 1.30 px |
| Very High | 151+    | #7f0000 (dark)   | 1.90 px |

### Other Layers

| Layer               | Symbol                  | Notes                        |
|---------------------|-------------------------|------------------------------|
| Distribution Points | Red square, 6pt         | Labelled with DP name        |
| Building Centroids  | Grey circle, 1.2pt      | Demand reference             |
| Hub Lines           | Grey line, 0.18pt, 45%  | Service allocation lines     |
| Routed Cables       | Blue #4393c3, 0.28pt    | Shortest-path routes         |

---

## File Structure

```
Fibre-Optic Distribution Network Analysis/
├── README.md
├── FibreOptic_Reusel.qgz                    QGIS project (3.x)
├── layers/
│   ├── Buildings_Residential.gpkg
│   ├── Roads_31831.gpkg
│   ├── Building_Centroids.gpkg
│   ├── Distribution_Points.gpkg
│   ├── Hub_Lines.gpkg
│   ├── Hub_Road_Intersections.gpkg
│   ├── Roads_Segmented.gpkg
│   ├── Trench_Network_Segmented.gpkg        cable_count field = main output
│   ├── Trench_Vertices.gpkg
│   └── Routed_Cables.gpkg
└── export/
    └── FibreOptic_Reusel_TrenchLoad.pdf     Final map output (A3, 200 dpi)
```

---

## Key Findings

- **552 residential buildings** identified as FTTH demand nodes after filtering.
- Trench network resolves to **2,334 fully segmented sections**.
- **1,767 segments carry active cable load** (76% of the network).
- Maximum cable concentration: **326 cables on a single segment** — highest-priority trunk.
- Three DPs allow load to be visualised by service zone.

---

## Software & Dependencies

- QGIS 3.x (tested on QGIS 3.34 LTR)
- Algorithms: native:reprojectlayer, native:centroids, native:splitwithlines,
  native:mergevectorlayers, native:lineintersections, native:extractvertices,
  native:shortestpathpointtolayer
- CRS: EPSG:25831 (ETRS89 / UTM Zone 31N)
- Data: © OpenStreetMap contributors

---
*Prepared for internal review — Reusel FTTH Expansion Network Viability Assessment*
