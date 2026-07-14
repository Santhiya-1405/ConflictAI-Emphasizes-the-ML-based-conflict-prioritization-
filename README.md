 ConflictAI: AI-Based Multi-Agent Path Finding (MAPF)

An interactive Multi-Agent Path Finding (MAPF) simulator that integrates **LaCAM**, **Conflict-Based Search (CBS)**, and a **Machine Learning (Random Forest)** model for intelligent conflict prioritization. The project provides real-time visualization, conflict detection, performance analysis, and benchmark evaluation for multi-agent navigation in grid-based warehouse environments.

---

 Project Overview

This project implements three different approaches for solving Multi-Agent Path Finding (MAPF):

- **LaCAM Solver**
- **Conflict-Based Search (CBS)**
- **Hybrid ML-CBS Solver**

Unlike traditional MAPF solvers, the Hybrid ML-CBS approach first generates independent shortest paths for every agent, detects all conflicts, extracts conflict features, and uses a trained Random Forest model to determine which conflict should be resolved first. CBS then resolves the selected conflict by replanning paths with constraints until a conflict-free solution is obtained.

The simulator also provides performance statistics, visualization, and automatic fallback mechanisms for difficult scenarios.

---

 Key Features

- Interactive warehouse/grid environment
- Dynamic obstacle creation
- Multiple agent support
- A* shortest path planning
- Vertex and Edge conflict detection
- LaCAM implementation using PIBT
- Complete CBS implementation
- Hybrid Machine Learning + CBS pipeline
- Random Forest conflict prioritization
- Automatic fallback to CBS
- Performance Analysis Dashboard
- Solution animation
- Graph visualization
- Synthetic dataset generation
- Random Forest model training

---

 🏗 Hybrid Solver Workflow

Warehouse/Grid
↓

Independent A* Path Generation

↓

Conflict Detection

↓

Feature Extraction

↓

Random Forest Prediction

↓

Highest Priority Conflict Selection

↓

CBS Constraint Generation

↓

Path Replanning

↓

Conflict Free?

├── No → Repeat

└── Yes → Final Solution

↓

Performance Analysis

---

 Available Solvers

## 1. LaCAM

LaCAM performs fast joint configuration search using Priority Inheritance with Backtracking (PIBT).

### Advantages

- Fast execution
- Scalable for many agents
- Efficient in sparse environments

### Limitation

If LaCAM cannot resolve deadlocks (such as adjacent swaps or cyclic movements), it automatically falls back to CBS.

---

## 2. Conflict-Based Search (CBS)

CBS is an optimal MAPF algorithm that resolves conflicts by creating constraints and replanning paths using A*.

### Advantages

- Optimal solution
- Complete algorithm
- Guarantees collision-free paths

### Limitations

- Higher runtime
- Larger memory usage
- Slower for many agents

---

## 3. Hybrid ML-CBS

The Hybrid Solver combines Machine Learning with CBS.

Workflow:

1. Generate independent shortest paths.
2. Detect all conflicts.
3. Extract conflict features.
4. Predict conflict priority using Random Forest.
5. Resolve the highest-priority conflict using CBS.
6. Repeat until no conflicts remain.
7. Fall back to full CBS if the greedy approach stalls.

---

🤖 Machine Learning Model

The project uses a **RandomForestRegressor** to assign a priority score to each detected conflict.

Each conflict is represented using the following features:

| Feature | Description |
|----------|-------------|
| Agent 1 ID | First conflicting agent |
| Agent 2 ID | Second conflicting agent |
| Conflict Time Step | Time of collision |
| Conflict X Coordinate | Row location |
| Conflict Y Coordinate | Column location |
| Conflict Type | Vertex or Edge |
| Nearby Obstacles | Obstacles within radius 2 |
| Nearby Agents | Agents within radius 2 |
| Current Makespan | Current maximum path length |
| Current Sum of Costs | Total path cost |

The conflict with the highest predicted priority is resolved first.

If the trained model (`rf_priority_model.joblib`) is unavailable, the simulator automatically switches to the built-in heuristic priority function.

---

📊 Performance Metrics

The simulator reports:

- Makespan
- Sum of Costs
- Runtime
- Solver Time
- Total Resolution Rounds
- Number of Conflicts
- Conflicts Resolved
- Average Path Length
- Average Waiting Time

---

🎮 Keyboard Controls

| Key | Function |
|------|----------|
| **1** | Obstacle Mode |
| **2** | Agent Placement Mode |
| **3** | Run LaCAM |
| **4** | Animate Solution |
| **5** | Run Hybrid ML-CBS |
| **G** | Toggle Graph Panel |
| **R** | Reset Grid |

---
📂 Project Structure

```
ConflictAI/
│
├── main.py                 # Main simulator
├── ui.py                   # User interface
├── config.py               # Configuration
├── color_utils.py          # Agent colors
│
├── astar.py                # Single-agent A*
├── distance_table.py       # BFS distance tables
├── pibt.py                 # Priority Inheritance
├── lacam.py                # LaCAM solver
├── cbs.py                  # CBS solver
│
├── conflict_detection.py   # Detect conflicts
├── feature_extraction.py   # ML feature extraction
├── ml_priority.py          # Random Forest prediction
├── hybrid_solver.py        # Hybrid ML-CBS solver
│
├── generate_dataset.py     # Dataset generation
├── train_model.py          # Train Random Forest
├── conflict_training_data.csv
├── rf_priority_model.joblib
│
└── requirements.txt
```

---

⚙ Installation

```bash
git clone https://github.com/yourusername/ConflictAI.git

cd ConflictAI

pip install -r requirements.txt
```

---

 ▶ Running the Project

Run the simulator

```bash
python main.py
```

Generate the training dataset

```bash
python generate_dataset.py
```

Train the Random Forest model

```bash
python train_model.py
```

---

📦 Requirements

- Python 3.10+
- pygame
- numpy
- pandas
- matplotlib
- scikit-learn
- joblib

Install all dependencies using:

```bash
pip install -r requirements.txt
```

---

🌍 Applications

- Warehouse Robot Navigation
- Autonomous Mobile Robots
- Smart Manufacturing
- Drone Fleet Coordination
- Airport Logistics
- Hospital Delivery Robots
- Industrial Automation
- Multi-Robot Systems

---

🔮 Future Enhancements

- Deep Reinforcement Learning
- Dynamic Obstacles
- ROS Integration
- 3D Warehouse Simulation
- Transformer-Based Conflict Prediction
- Real Robot Deployment

---


📚 References

1. Sharon, G., Stern, R., Felner, A., & Sturtevant, N. R. (2015). Conflict-Based Search for Optimal Multi-Agent Path Finding.

2. Okumura, K. et al. (2023). LaCAM: Search-Based Multi-Agent Path Finding.

---

