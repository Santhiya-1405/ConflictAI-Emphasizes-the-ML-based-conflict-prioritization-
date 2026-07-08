ConflictAI (Emphasizes the ML-based conflict prioritization)

An interactive Multi-Agent Path Finding (MAPF) simulator with three solvers
available side by side:

- **LaCAM** (`lacam.py` + `pibt.py`) - fast joint-configuration search, with
  automatic fallback to CBS if it livelocks (known PIBT weakness on
  adjacent-swap / rotational deadlocks).
- **CBS** (`cbs.py`) - complete conflict-based search, used both as LaCAM's
  fallback and as the hybrid pipeline's fallback.
- **Hybrid ML-CBS** (`hybrid_solver.py`) - the pipeline below, new in this
  version.

## The hybrid pipeline

```
Warehouse/Grid
      |
      v
    LaCAM              (fast, per-agent initial paths - see note below)
      |
      v
Initial Paths
      |
      v
Conflict Detection      (conflict_detection.py: detect_all_conflicts)
      |
      v
Feature Extraction      (feature_extraction.py: 10 features per conflict)
      |
      v
Random Forest            (ml_priority.py: rf_priority_model.joblib)
      |
      v
Conflict Priority Prediction
      |
      v
CBS Conflict Resolution  (constrain + A* replan the higher-priority side)
      |
      v
Updated Paths
      |
      v
Conflict Free? --No--> back to Conflict Detection
      |
     Yes
      |
      v
Final Solution
      |
      v
Performance Analysis     (rounds, conflicts resolved, cost, makespan, timing)
```

**Note on the "LaCAM" stage:** LaCAM's own PIBT-based search already
guarantees conflict-free joint paths when it succeeds, so a second,
downstream conflict-resolution stage would have nothing left to do. To make
the pipeline meaningful, the hybrid solver's "Initial Paths" stage instead
gives every agent its own independent shortest path (ignoring every other
agent) - fast, but deliberately conflict-prone, which is exactly what the
Random-Forest-guided CBS stage exists to clean up.

### Random Forest: what it predicts, and how it was trained

At every round, `detect_all_conflicts()` finds *every* conflict at the
earliest colliding timestep. Each one gets turned into a 10-feature vector:

| # | Feature                 | Meaning                                             |
|---|--------------------------|------------------------------------------------------|
| 1 | `agent1_id`              | Agent 1 ID                                            |
| 2 | `agent2_id`              | Agent 2 ID                                            |
| 3 | `conflict_timestep`      | Conflict Time Step                                    |
| 4 | `conflict_x`             | Conflict X Coordinate (row)                           |
| 5 | `conflict_y`             | Conflict Y Coordinate (col)                           |
| 6 | `conflict_type`          | Conflict Type (0 = vertex, 1 = edge)                  |
| 7 | `nearby_obstacles`       | Number of Nearby Obstacles (radius 2)                 |
| 8 | `nearby_agents`          | Number of Nearby Agents (radius 2, at that timestep)  |
| 9 | `current_makespan`       | Current Makespan                                      |
| 10| `current_sum_of_costs`   | Current Sum of Costs                                  |

The Random Forest (`RandomForestRegressor`) scores each candidate conflict;
the highest-scoring one is resolved first (both agents' branches are tried
via A*, and the cheaper resulting path set is kept - same idea as CBS
branching, just one branch expanded per round instead of a full search
tree, which is what makes this fast enough for an interactive simulator).

True "this was the optimal conflict to resolve first" labels would require
exhaustively searching every possible CBS branching order, which is
combinatorially expensive to generate at scale. Instead, `generate_dataset.py`
labels each conflict with a transparent, hand-designed heuristic
(`heuristic_priority_score` in `ml_priority.py`): edge conflicts score
higher than vertex conflicts, congestion (nearby agents/obstacles) raises
the score, and conflicts happening sooner are weighted more urgent than
ones far in the future - plus Gaussian noise so the model learns a smooth,
generalizable function of the features rather than memorizing the formula.
The trained model reached **R² ≈ 0.85** on a held-out test split (see
`train_model.py` output). If no trained model file is present,
`ml_priority.py` transparently falls back to that same heuristic formula,
so the simulator still runs correctly - it just isn't using the learned
model yet.

If the greedy round-by-round loop stalls (the same pair of agents keeps
conflicting - typically one agent parked on its goal cell repeatedly
blocking another) or exceeds its round budget, the pipeline falls back to
full CBS for a guaranteed-complete answer, mirroring the existing
LaCAM -> CBS fallback pattern. Full CBS itself also has a 15-second safety
budget (`DEFAULT_MAX_SECONDS` in `cbs.py`) so a pathologically congested
scenario can't freeze the UI.

## Running it

```bash
pip install -r requirements.txt

# (one-time, or whenever you want to retrain) build training data + model:
python generate_dataset.py
python train_model.py

# run the simulator:
python main.py
```

A pretrained `rf_priority_model.joblib` is already included, so `main.py`
works immediately without the two training steps above.

## Controls

| Key | Action |
|-----|--------|
| `1` | Obstacle mode - click cells to toggle walls |
| `2` | Start/Goal mode - click to place a new agent |
| `3` | Run LaCAM (falls back to CBS if it can't solve) |
| `4` | Animate the current solution |
| `5` | Run Hybrid ML-CBS (this pipeline) - shows a Performance Analysis panel |
| `G` | Toggle the path-length graph panel |
| `R` | Reset the grid and agents |

## Files

| File                      | Role                                                              |
|---------------------------|--------------------------------------------------------------------|
| `config.py`                | Grid/window/color constants, search-bound helpers                  |
| `astar.py`                 | Constrained single-agent A* (used by CBS and the hybrid solver)     |
| `distance_table.py`        | BFS distance tables (used by PIBT)                                  |
| `pibt.py`                  | Priority Inheritance with Backtracking (LaCAM's low level)          |
| `lacam.py`                 | LaCAM high-level search + CBS fallback                              |
| `cbs.py`                   | Conflict-Based Search (with a time-budget safety net)                |
| `conflict_detection.py`    | `detect_all_conflicts` + `make_constraint` for the hybrid pipeline  |
| `feature_extraction.py`    | The 10-feature vector per conflict                                   |
| `generate_dataset.py`      | Synthetic scenario + label generator -> `conflict_training_data.csv`|
| `train_model.py`           | Trains + saves `rf_priority_model.joblib`                            |
| `ml_priority.py`           | Loads the model, scores conflicts, picks the priority one            |
| `hybrid_solver.py`         | The full pipeline shown above                                       |
| `color_utils.py`           | Per-agent color generation                                            |
| `ui.py`                    | Pygame rendering, including the Performance Analysis panel           |
| `main.py`                  | Event loop / entry point                                             |
