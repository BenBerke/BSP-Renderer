# 2.5D BSP Software Engine

A retro pseudo-3D engine written in C++ from scratch. It uses standard software rendering techniques and a **Binary Space Partitioning (BSP) Tree** to resolve rendering order and handle dynamic spatial queries for entities.

---

## Features

* **Portal-Based Sector Rendering:** Walls act as portals linking sectors with varying floor heights and ceiling clearances.
* **BSP-Driven Visibility:** Implements a full Binary Space Partitioning tree build sequence (`BuildBSP` / `BuildSubSectors`) to eliminate depth sorting performance bottlenecks by traversing static level geometry deterministically based on player position.
* **Point-in-Polygon Collision & Queries:** Real-time spatial tracking (`FindSector`) allows immediate evaluation of the current player context based on coordinate arrays.
* **Custom Software Framebuffers:** Built completely overhead-free without reliance on modern GPU graphics APIs (OpenGL/DirectX), pushing rasterized data to a screen layer managed via raw multi-buffered frameworks.

---

## Technical Level Architecture

The code defines a custom 4-quadrant layout broken down below:

```text
       [-220, 160]               [0, 160]               [220, 160]
            +-----------------------+-----------------------+
            |                       |                       |
            |       SECTOR 0        |       SECTOR 1        |
            |     (Top-Left)        |     (Top-Right)       |
            |  Floor:0 / Ceil:48    |  Floor:8 / Ceil:48    |
            |                       |                       |
 [-220, 0]  +-----------------------+-----------------------+ [220, 0]
            |                       |                       |
            |       SECTOR 2        |       SECTOR 3        |
            |    (Bottom-Left)      |    (Bottom-Right)     |
            |  Floor:0 / Ceil:32    |  Floor:-8 / Ceil:40   |
            |                       |                       |
            +-----------------------+-----------------------+
       [-220, -160]              [0, -160]              [220, -160]
```

### Sector Configurations

| Sector ID | Spatial Description | Floor Height | Ceiling Height | Unique Property |
|------------|--------------------|---------------|----------------|----------------|
| **Sector 0** | Top-Left Room | `0.0f` | `48.0f` | Baseline/Normal Space |
| **Sector 1** | Top-Right Room | `8.0f` | `48.0f` | Raised Floor Step |
| **Sector 2** | Bottom-Left Room | `0.0f` | `32.0f` | Lowered Ceiling Area |
| **Sector 3** | Bottom-Right Room | `-8.0f` | `40.0f` | Sunken Pool/Pit |

---

## Main Run Loop Architecture

The system coordinates initialization, static construction pipelines, and updates inside an independent thread loop:

```text
 [Initialize Engine Systems] ──> [Assemble Map Vectors] ──> [Compile Static BSP Tree]
                                                                      │
                                                                      ▼
  [Render Context Frame] <── [Update Player & Physics] <── [Poll Input & Delta Time]
```

1. **Map Loading:** Binds sequential arrays for processing directly via `MapEditor::CreateWallDirectly` and `MapEditor::CreateSectorDirectly`.
2. **BSP Compilation:** `BuildBSP` recursively evaluates lines to construct spatial sorting leaves. `BuildSubSectors` then injects polygonal data boundaries into matching structural branches.
3. **Runtime Iteration Loop:**
   * Polls basic keyboard scanning arrays (`InputManager`).
   * Tracks structural frames per second (`GameTime`).
   * Traces player motion indices against boundaries to safely calculate wall slide physics.
   * Traverses structural BSP branches using the player's 2D coordinate vector to update context records.
   * Offloads computed horizontal slices onto the active display layer (`Renderer`).

---

## File Structure Hierarchy

The project expects header files grouped into specific directories:

```text
├── Headers/
│   ├── Engine/
│   │   ├── InputManager.h    # SDL Scan code mapping inputs
│   │   └── GameTime.h        # Frame calculations and clock monitoring
│   ├── Objects/
│   │   ├── Player.h          # Player structures, positioning data, and physics movement logic
│   │   └── Wall.h            # Geometric wall definitions including front/back sector bindings
│   └── Renderer/
│       ├── MapEditor.h       # Level building methods
│       ├── Renderer.h        # Fixed aspect resolution pipeline (960x600)
│       └── BSP.h             # Spatial structural compilation definitions
└── main.cpp                  # Engine entrypoint & map instantiation
```
