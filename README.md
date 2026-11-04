# 2E-S-MC-LRP-EV: Two-Echelon Stochastic Location-Routing with Fleet Electrification

[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![Optimization](https://img.shields.io/badge/solver-Gurobi%2011%2B-orange.svg)](https://www.gurobi.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Paper: TRD](https://img.shields.io/badge/Journal-Transportation%20Research%20Part%20D-green.svg)](https://www.sciencedirect.com/journal/transportation-research-part-d-transport-and-environment)

Official repository for the research paper:  
**"A Two-Echelon Stochastic Location-Routing Problem with Fleet Electrification for Sustainable and Equitable Waste Management: A Case Study in Brazil"**  
*David Gabriel de Barros Franco, Thiago Alves de Queiroz, Giulia Caselli, and Manuel Iori.*

---

## 📌 Overview

This repository provides an end-to-end, data-driven Operations Research (OR) framework designed to solve the **Two-Stage Stochastic Multi-Commodity Two-Echelon Location-Routing Problem with Electric Vehicles and Synchronization (2E-S-MC-LRP-EV)**. 

The framework redesigns municipal solid waste (MSW) reverse logistics networks by coupling:
- **First Echelon (E1):** Capillary, high-frequency collection in dense street networks using light-duty Electric Vehicles (EVs, 1.2 t payload).
- **Intermediate Decoupling Layer:** Multi-compartment smart bins acting as physical and temporal consolidation buffers.
- **Second Echelon (E2):** Consolidated arterial routing to peripheral transfer stations/landfills via heavy-duty electric compactor trucks (15.0 t payload).

The pipeline was benchmarked against the single-echelon diesel baseline across **37 real-world municipal street networks** in the State of Tocantins, Brazil.

┌────────────────────────────────────────────────────────────────────────────────────────┐
│                                 DECISION PIPELINE                                      │
├─────────────────────┬──────────────────────┬────────────────────┬──────────────────────┤
│  Phase I: Data      │  Phase II & III:     │  Phase IV & V:     │  Phase VI:           │
│  Mining & GBM       │  Baseline & Arc-     │  ALNS Warm-Start & │  Physics Recalculation│
│  Simulation         │  Districting         │  Branch-and-Cut    │  & DCF Economics     │
│                     │                      │                    │                      │
│ • OSMnx topology    │ • Clarke-Wright      │ • Regret-k ALNS    │ • WTW LCA Emissions  │
│ • Overture Maps     │   Baseline           │ • Sector sub-MILPs │ • Phased 10-Yr CAPEX │
│ • Moran's I test    │ • Priority Queue     │ • Lazy DFJ cuts    │ • Circular Economy   │
│ • Monte Carlo GBM   │ • Topological contig.│ • Minimax Equity   │ • EVPI Hedge proxy   │
└─────────────────────┴──────────────────────┴────────────────────┴──────────────────────┘

---

## ✨ Key Features & Methodological Innovations

1. **Topology-Aware Arc-Districting:** Replaces arbitrary Euclidean Voronoi/K-Means clustering with a capacitated region-growing algorithm over the road network's *line-graph*, guaranteeing that street segments are never bisected between operational zones.
2. **Dual-Risk Scenario Structure:** Formulates a two-stage stochastic program where the *Robust-Peak* scenario compounds long-term macroeconomic volatility (95th-percentile GBM Monte Carlo path) with the operational 3-day weekend collection interval.
3. **Branch-and-Cut accelerated by ALNS:** Generates high-quality warm starts via an Adaptive Large Neighborhood Search metaheuristic and dynamically injects exponential Dantzig-Fulkerson-Johnson (DFJ) subtour elimination constraints via solver callbacks.
4. **Thermodynamic Payload-Dependent Energy Physics:** Disaggregates tare and payload continuous consumption for both electric and conventional fleets, evaluated under a Well-to-Wheel (WTW) Life-Cycle Assessment calibrated to the Brazilian hydroelectric grid.
5. **Equity & Environmental Justice:** Integrates a scenario-pooled minimax equity constraint bounding the worst-case EV round-trip distance, verified via post-optimality Pareto duality approximations.

---

## 🗂 Repository Structure

├── data/
│   ├── municipios_data.txt          # Demographic, waste, and spatial bounding boxes for 37 cities
│   └── cache/                       # Cached OSMnx graphs (.graphml) and building footprints (.gpkg)
├── src/
│   ├── open_lrp_new_33_0_tocantins.py # Main monolithic execution script
│   └── modules/                     # Modular components (ALNS, districting, physics, DCF)
├── instances/                       # Extracted municipal benchmark graph instances
├── results/                         # Output Excel sheets (Executive Summary, Deployment, EVPI)
├── figures/                         # High-resolution generated analytical panels (1 to 5)
├── requirements.txt                 # Python dependencies
├── LICENSE                          # MIT License
└── README.md                        # Documentation

---

## ⚙️ Requirements & Installation

### Prerequisites
- **Python:** 3.10 or higher
- **Optimization Solver:** [Gurobi Optimizer](https://www.gurobi.com/) (version 10.0+ or 11.0+). A valid Gurobi license (Academic, Cloud, or Commercial) is required.

### 1. Clone the repository
git clone https://github.com/your-username/2E-S-MC-LRP-EV.git
cd 2E-S-MC-LRP-EV

### 2. Create a virtual environment
# Using conda (recommended for geospatial libraries)
conda create -n lrp_ev python=3.10 -y
conda activate lrp_ev

# Or using venv
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

### 3. Install dependencies
pip install --upgrade pip
pip install -r requirements.txt

> **Note on Geospatial Dependencies:** If installing via `pip` on Windows, ensure that `GDAL`, `Fiona`, and `pyproj` are properly configured. Installing via `conda install -c conda-forge osmnx contextily overturemaps` avoids binary compilation issues.

---

## 🚀 Usage

### Single-Municipality Execution
To run the optimization for a specific municipality (e.g., *Divinópolis do Tocantins* or *Palmas*), set the target city inside `src/open_lrp_new_33_0_tocantins.py` or run:

python src/open_lrp_new_33_0_tocantins.py --municipality "Divinópolis do Tocantins"

The execution flow automatically performs:
1. Online ingestion and local caching of OpenStreetMap topology and Overture Maps building footprints.
2. Dasymetric demand disaggregation and Moran's $I$ spatial autocorrelation verification.
3. Baseline Clarke-Wright single-echelon heuristic routing.
4. Line-graph capacitated districting into $K$ macro-sectors.
5. ALNS warm-start seeding and exact Branch-and-Cut resolution in Gurobi.
6. Decadal discounted cash flow (NPV, TCO, EVPI) and high-resolution figure plotting.

### Batch Benchmark Execution (All 37 Municipalities)
To execute the complete state-wide benchmark across all 37 municipal instances:

python run_batch_benchmark.py --input data/municipios_data.txt --output_dir results/

---

## 📊 Summary of Benchmark Results (37 Municipalities)

Across the 37 municipal case studies in Tocantins, Brazil, the 2E-EV system demonstrated transformative operational, environmental, and fiscal improvements over the single-echelon diesel compactor baseline:

| Metric | Mean Savings vs. Baseline | Median Savings | Wilcoxon Test ($p$-value) |
| :--- | :---: | :---: | :---: |
| **Life-Cycle GHG Emissions (WTW)** | **+94.6%** | **+95.0%** | $p = 1.455 \times 10^{-11}$ $^{***}$ |
| **10-Year NPV Total Cost of Ownership (TCO)** | **+30.8%** | **+38.4%** | $p = 4.467 \times 10^{-09}$ $^{***}$ |
| **Cumulative 10-Year Revenue Growth** | **+62.3%** | **+62.3%** | $p = 1.455 \times 10^{-11}$ $^{***}$ |
| **Total Daily Travel Distance (km)** | **+26.4%** | **+36.8%** | $p = 1.306 \times 10^{-04}$ $^{***}$ |
| **Heavy-Duty Compactor Fleet Sizing** | **-60.0%** (fewer trucks) | **-62.5%** | $p = 2.477 \times 10^{-07}$ $^{***}$ |
| **Expected Value of Perfect Information (EVPI)** | **11.1%** of OPEX | **11.8%** of OPEX | — |

*Significance: $^{***}\; p < 0.001$. Full instance-by-instance tables are available in `results/` and Appendix D of the paper.*

---

## 📈 Analytical Output Panels

When executed, the pipeline exports 5 publication-ready, 300-DPI multi-panel graphics into the `figures/` directory:

1. **`Panel_1_Analytics.png`:** 10-year Monte Carlo Geometric Brownian Motion demand projections and Well-to-Wheel decarbonization trajectories.
2. **`Panel_2_Geospatial.png`:** Topological street-anchored macro-zoning and Gaussian Kernel Density Estimation (KDE) waste heatmaps.
3. **`Panel_3_Topologies.png`:** Comparative routing topologies (Baseline single-echelon door-to-door vs. 2E-EV Hamiltonian compactor tours and EV capillary allocations).
4. **`Panel_4_Profit_Carbon.png`:** Operating cash flow breakdown and Cap-and-Trade carbon penalty surplus/deficit dynamics.
5. **`Panel_5_BiLevel_Economics.png`:** Circular economy diversion sensitivity, Equity vs. Profit Pareto frontiers, and decadal Payback/Break-even cash curves.

---

## 📜 Citation

If you use this framework, dataset, or code in your research, please cite our paper:

@article{Franco2026_2ELRP,
  author    = {David Gabriel de Barros Franco and Thiago Alves de Queiroz and Giulia Caselli and Manuel Iori},
  title     = {A Two-Echelon Stochastic Location-Routing Problem with Fleet Electrification for Sustainable and Equitable Waste Management: A Case Study in Brazil},
  journal   = {Transportation Research Part D: Transport and Environment},
  year      = {2026},
  note      = {Special Issue: Electrifying heavy-duty trucks and buses}
}

---

## 🤝 Acknowledgments

This research received financial support from:
- **FAPEG** (State of Goiás Research Foundation)
- **CNPq** (National Council for Scientific and Technological Development, Brazil) — Grant numbers: `408722/2023-1`, `315555/2023-8`, `444679/2024-3`, `407005/2024-2`, `409750/2024-7`, and `200286/2026-9`.
- **PNRR-TNE Project:** *Green & Pink for Sustainable Education (GPSEducation)* (Project Proposal `TNE23-00012`, CUP `D74G23000280006`), funded by the European Union NextGenerationEU (Italian National Recovery and Resilience Plan - PNRR).

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
