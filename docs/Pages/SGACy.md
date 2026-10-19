---
title: SGACy
description: Spatial Graph Analysis in Cython
---

# Spatial Graph Analysis in Cython

SGACy is a specially developed library to perform efficient pathfinding and other related functions on a spatial graph using Cython. This level of optimization enables for Urban Network Analysis (UNA) in the scale of cities on an acceptable time/processing cost. Comparison tests with networkx results in 20-100x times faster for single pathfinding, and up to 3 to 4 magnitudes on alternative-pathfinding, which are crucial for detour ratios on some UNA's toolset.

SGACy can be operated indepedently from SNAPy, some simplified data interaction are already supported internally (i.e. input data from geopandas). It has dependency on geopandas and numpy, for some minor operations and also as several's function output format for better integration with SNAPy. If possible, future development may result in SGACy being an independent library, with more focused functionality towards general spatial graph operations; Since SNAPy's operations are specific towards Urban Network Analysis calculations.

SGACy's functionality is split into two types and scripts, with:
- graph.pyx
    Handles the main class GraphCy for graph data and operations.
- geom.pyx
    Handles geometry related preparations.

Prebuilt extension modules shipped in `SNAPy/SGACy`:

- Windows: `graph.cp311-win_amd64.pyd`, `graph.cp312-win_amd64.pyd` (and matching `geom`)
- Linux x86_64: `graph.cpython-311-x86_64-linux-gnu.so`, `graph.cpython-312-x86_64-linux-gnu.so` (and matching `geom`)
- macOS Darwin arm: `graph.cpython-312-darwin.so` (and matching `geom`)

Linux binaries are compiled in Docker against Debian bookworm glibc. From `SNAPy/SGACy`:

```powershell
.\build_linux.ps1
```

or a single CPython:

```powershell
docker build --build-arg PY=3.11 -t sgacy-linux-311 .
docker run --rm -v "${PWD}:/src" sgacy-linux-311
```

#### GraphCy <i>class</i>

:   Main class of SGACy that contains most functionality of network/graph analysis.
    To provide effective spatial network related operations, edges geometry are abstracted to only graph values, but still maintains nodes/endpoints coordinates for indexing values or heurestic capabilities (i.e. Astar operations). 

!!! example "Work in Progress"
    Contents of this page is still in progress
    

<br><br>
@October2026
