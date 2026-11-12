# A* vs Greedy Best-First Search: Route Finding in Dhaka

A Google Colab notebook that builds a real road graph of Bangladesh from OpenStreetMap data and compares two search algorithms, **A\*** and **Greedy Best-First Search**, on a driving route from **Dhaka University** to **Hazrat Shahjalal International Airport**.

## Overview

| | |
|---|---|
| **Start** | Dhaka University (23.7330, 90.3920) |
| **Goal** | Hazrat Shahjalal Int'l Airport (23.8433, 90.3978) |
| **Graph** | Bangladesh driving network, 643,202 nodes and 1,542,061 edges |
| **Edge cost** | Road segment length in metres |
| **Heuristic** | Haversine (straight-line) distance, used by both algorithms |

The haversine heuristic never overestimates real road distance (it is admissible), so A\* is guaranteed to return the shortest path. Both algorithms use the same heuristic, so the comparison is fair.

## Results

| Metric | A\* | Greedy BFS |
|---|---|---|
| Distance | 14.297 km | 21.197 km |
| Nodes in path | 127 | 196 |
| Run time | 0.0871 s | 0.0142 s |
| Shortest route? | Yes | No |

Greedy is about 6× faster but its route is about 6.9 km (48%) longer. Run times vary by machine, so treat them as approximate.

## Notebook structure

The notebook has two sections, each a complete, independent pipeline:

1. **A\* Search**
2. **Greedy BFS**

Each section follows the same steps:

1. **Install** dependencies.
2. **Download** `bangladesh-latest.osm.pbf` from Geofabrik (streamed in 1 MB chunks, skipped if the file already exists).
3. **Build the graph** with `pyrosm` (`network_type="driving"`, NetworkX graph). This is the slowest step, about 6 minutes.
4. **Snap** the start and goal GPS points to the nearest road nodes using a `BallTree` with haversine distance.
5. **Run the search**:
   - A\*: `nx.astar_path(G, start, goal, heuristic=heuristic, weight="length")`
   - Greedy: a custom `heapq` priority queue ordered by `h(n)` only
6. **Report** path status, node count, total distance (m and km) and execution time.
7. **Visualise** the route on an interactive Folium map with start and destination markers and an info box.

## Requirements

- Python 3 (tested on Google Colab)
- `pyrosm`
- `networkx`
- `folium`
- `scikit-learn`
- `numpy`
- `requests`
- `shapely` (used in the Greedy map cell; installed with `pyrosm`)

Install everything with:

```bash
pip install -q pyrosm networkx folium scikit-learn
```

## How to run

1. Open the notebook in Google Colab.
2. Run the cells in the **A\* Search** section from top to bottom.
3. Run the cells in the **Greedy BFS** section from top to bottom.
4. The Folium map appears as the output of the last cell in each section.

The dataset is saved to `/content/bangladesh.osm.pbf`. Change `file_path` if you run outside Colab. The Bangladesh extract is large, so make sure there is enough disk space and RAM. Building the graph for the whole country takes several minutes.

## Customising

To test a different route, change these in the notebook:

```python
start_location = (23.7330, 90.3920)   # (lat, lon)
goal_location  = (23.8433, 90.3978)   # (lat, lon)
```

## Limitations

- Cost is distance only. There is no traffic, speed limit or travel-time data.
- Only one start and goal pair was tested.
- Loading the full country graph is slow. Cropping the map to Dhaka would speed this up.
- Run times depend on the machine.

## Future work

- Add Dijkstra as an exact baseline.
- Use travel time instead of length as the edge weight.
- Test many random start and goal pairs.
- Count nodes expanded by each algorithm to compare search effort.

## Data and credits

- Road data: [OpenStreetMap](https://www.openstreetmap.org) extract from [Geofabrik](https://download.geofabrik.de/asia/bangladesh.html), licensed under ODbL. © OpenStreetMap contributors.
- Map tiles: Esri World Street Map.
