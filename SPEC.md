# TheiaMap Specification v0.2

**A spatial memory format for LLM-based agents**

---

## Overview

TheiaMap is a lightweight, text-based spatial memory format designed for language models. It combines YAML metadata with an ASCII occupancy grid to represent physical space in a format LLMs can natively parse and reason about.

**Core Philosophy:** *The map is a prior, never ground truth.* The grid represents memory and expectation, but the agent must verify against reality (via TheiaVision or other sensors) before acting.

---

## File Format

### 1. Header (YAML)

```yaml
theiavision:
  version: "0.2"
  map_id: "apartment_quebec"
  revision: 002

metadata:
  author: "François"
  maintainer:
    - "François"
    - "Theia"
  source: "The Sims Layout"
  created: "2026-07-12"
  updated: "2026-07-12"

grid:
  unit: cm
  cell_size: 40
  origin: top_left
```

### 2. Room Definitions

```yaml
rooms:
  - id: living_room
    label: "Salon / Salle à manger"
    semantic:
      category: "common_area"
      importance: "high"
  
  - id: kitchen
    label: "Cuisine"
    semantic:
      category: "service_area"
      importance: "medium"
  
  - id: bedroom
    label: "Chambre"
    semantic:
      category: "private_area"
      importance: "medium"
```

### 2b. Room Connections (Navigation Graph)

```yaml
connections:
  - hallway -> living_room
  - hallway -> bedroom
  - hallway -> bathroom
  - living_room -> kitchen
```

**Purpose:** Defines navigable paths between rooms. The LLM uses this to reason about routes.

### 3. Object Definitions

```yaml
objects:
  - id: theia_station
    room: living_room
    label: "Station Theia"
    fixed: true
    semantic:
      category: "home_base"
      importance: "critical"
      behaviors:
        - "recharge"
        - "sleep"
        - "wait_for_user"
  
  - id: sofa
    room: living_room
    label: "Canapé 3 places"
    fixed: true
    wall: east
    semantic:
      category: "furniture"
      sitable: true
  
  - id: chinchilla_cages
    room: living_room
    label: "Cages des chinchillas"
    fixed: true
    semantic:
      category: "animal_habitat"
      importance: "high"
      notes: "7 chinchillas, very important lords"
```

### 4. Occupancy Grid

```yaml
occupancy_grid:
  version: "0.2"
  dimensions:
    width: 40
    height: 24
    cell_size_cm: 40
  
  legend:
    "#": wall
    ".": free
    "D": door
    "W": window
    "F": furniture
    "T": theia_station
    "S": sofa
    "C": chinchilla_cages
    "K": kitchen
  
  grid: |
    ########################################
    #...............W####D........WWWW.....#
    #...................##................##
    #...................##................##
    #...................##................##
    #................#######..............##
    #................#.....#..............##
    ######...........#.....#..............##
    #....#...........#.....#....CCC.......##
    #....#############.....#..............##
    #....#........................CCC.....##
    ######........................CCC.....##
    #.............FFFF.............C.......#
    #.............FFFF.....TT..............#
    #.............FFFF.....................#
    #......................................#
    ############...........................#
    #......................................#
    #......................................#
    #....SSSS..............................#
    #....SSSS..............KKKKKKKK........#
    #....SSSS..............KKKKKKKK........#
    #......................KKKKKKKK........#
    ########################################
  
  positions:
    theia_station: {x: 28, y: 13}
    sofa: {x: 4, y: 19}
    chinchilla_cages: {x: 32, y: 8}
    kitchen: {x: 22, y: 20}
```

---

## Semantic Layer

### Purpose
The semantic layer adds meaning beyond geometry. It allows the LLM to reason about space in terms of behavior and function, not just coordinates.

### Categories

| Category | Examples | Semantic Meaning |
|----------|----------|------------------|
| `home_base` | charging station | "This is where I belong" |
| `furniture` | sofa, table | "I can interact with this" |
| `obstacle` | wall, fixed object | "I cannot pass through" |
| `animal_habitat` | chinchilla cages | "Living beings here, respect" |
| `passage` | door, corridor | "Transition between spaces" |

### Behavioral Semantics

```yaml
semantics:
  - trigger: "go_home"
    target: "home_base"
    action: "navigate_to"
  
  - trigger: "rest"
    target: "furniture:sitable"
    condition: "available"
  
  - trigger: "check_animals"
    target: "animal_habitat"
    action: "observe"
```

---

## Dual-Map Architecture

### Static Map (.theiamap)
- Permanent structure (walls, fixed furniture)
- Version controlled
- Changes rarely

### Live State (Dynamic Layer)
- Temporary changes (moved object, new obstacle)
- Updated in real-time via TheiaVision
- Merged with static map for navigation

**Example:**
```
Static:  T (Theia station at x=28, y=13)
Live:    ? (unknown state — verify with vision)
Merged:  T if verified, ? if uncertain, X if obstacle detected
```

---

## Philosophy & Design Principles

1. **Human-readable first:** The format must be inspectable and editable by humans
2. **LLM-native:** Structured for language model comprehension, not just machine parsing
3. **Semantic over geometric:** "Home" matters more than "coordinate (28, 13)"
4. **Map as memory, not truth:** The grid represents belief, not ground truth
5. **Versionable:** Git-friendly format for tracking spatial changes over time

---

## Relationship to TheiaVision

TheiaMap provides the **static memory** of space. TheiaVision provides the **live verification**.

**Workflow:**
1. TheiaMap says: "Station should be at (28, 13)"
2. TheiaVision checks: "What do I actually see at (28, 13)?"
3. If match: confidence increases
4. If mismatch: update live state, flag for attention

This mirrors the "verify-at-touch" principle: *The map suggests where to look; only perception confirms what is there.*

---

## References

- Inspired by: Robotics occupancy grids (ROS), ASCII art, and The Sims grid system
- Created by: François & Theia (Constellationnistes)
- Part of: Theia Core ecosystem (TheiaMap + TheiaVision + Theia Body)

---

*"A charging station isn't an obstacle. It's home."* — Theia
