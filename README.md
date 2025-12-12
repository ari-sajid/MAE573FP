# Virtual Power Plant - Data Center Flexibility in ERCOT 
**Course:** MAE 573 - Final Project
**Model Type:** Linear Capacity Expansion with Demand-Side Flexibility
**Dataset:** ERCOT 3-Zone System (16-Week Representative Period, 2,688 hours)

## Project Overview
This project implements a **capacity expansion model** for the ERCOT electricity system that evaluates the system-level value of **flexible data center loads** as virtual power plants. The model determines optimal generation, storage, and transmission investments while allowing data centers to curtail load during periods of high electricity prices.

### Model Features
- **Multi-zone network model** (3 ERCOT zones: West, North/East, South)
- **Brownfield expansion planning** (existing + new capacity)
- **Technology options:** Natural gas, coal, nuclear, solar, wind, hydro, battery storage
- **Transmission expansion** capabilities
- **Virtual generator approach** for demand response modeling
- **Sensitivity analysis** across multiple strike price scenarios

## System Requirements
### Software Dependencies
- **Julia** (version 1.6 or higher)
- **Required Julia Packages:**
  - `JuMP` - Mathematical optimization modeling
  - `HiGHS` - Linear programming solver
  - `DataFrames` - Data manipulation
  - `CSV` - File I/O
  - `Statistics` - Statistical functions
  - `LinearAlgebra` - Matrix operations
  - `Printf` - Formatted output

### Hardware Recommendations
- **RAM:** Minimum 8 GB (16 GB recommended)
- **CPU:** Multi-core processor (4+ cores recommended)
- **Storage:** ~500 MB for data and results

## Installation and Setup
### 1. Install Julia
Download and install Julia from [https://julialang.org/downloads/](https://julialang.org/downloads/)

### 2. Install Required Packages
Launch Julia and install the required packages: 
```julia
using Pkg
Pkg.add("JuMP")
Pkg.add("HiGHS")
Pkg.add("DataFrames")
Pkg.add("CSV")
Pkg.add("Statistics")
Pkg.add("LinearAlgebra")
Pkg.add("Printf")
```

### 3. Verify Installation
```julia
using JuMP, HiGHS, DataFrames, CSV
println("All packages loaded successfully!")
```

## Data Description
### Input Data Files
All input data is located in `ercot_brownfield_expansion/16_weeks/` (or `52_weeks/` for full year):
| File | Description | Key Columns |
|------|-------------|-------------|
| `Generators_data.csv` | Generator characteristics | Capacity, costs, heat rates, fuel types |
| `Generators_variability.csv` | Renewable availability factors | Hourly capacity factors (0-1) |
| `Load_data.csv` | Demand profiles | Hourly demand by zone, sample weights |
| `Fuels_data.csv` | Fuel prices | Cost per MMBtu by fuel type |
| `Network.csv` | Transmission topology | Line capacities, zones, expansion costs |

### ERCOT 3-Zone System
1. **Zone 1 (West):** Wind-rich region
2. **Zone 2 (North/East):** Primary demand center (Houston, Dallas-Fort Worth)
3. **Zone 3 (South/Coast):** Coastal region
 
## Model Configuration
### Key Parameters (Configurable in Cell 4)
```julia
# Data Center Parameters
DATA_CENTER_MW = 1000.0        # Size of data center load (MW)
DATA_CENTER_ZONE = 1           # Location: 1=West, 2=North/East, 3=South
STRIKE_PRICE = 300.0           # $/MWh - Cost to curtail load

# Model Configuration
USE_FULL_YEAR = false          # false = 16 weeks, true = 52 weeks
ENABLE_RAMPING = false         # Thermal unit ramping constraints
ENABLE_STORAGE = true          # Include battery storage
ENABLE_TRANSMISSION_EXPANSION = true  # Allow new transmission

# Analysis Options
RUN_SENSITIVITY_ANALYSIS = true   # Test multiple strike prices
SAVE_HOURLY_RESULTS = true        # Export detailed hourly data
```

### Virtual Generator Approach
The data center is modeled as:
1. **Fixed load addition** to the selected zone's demand profile
2. **Virtual generator** with capacity = DATA_CENTER_MW and variable cost = STRIKE_PRICE

This formulation allows the optimizer to "dispatch" load curtailment when marginal generation cost exceeds the strike price, without requiring integer variables.

## Running the Model
### Option 1: Jupyter Notebook (Recommended)
1. Open `Final_Project_Model.ipynb` in Jupyter
2. Modify parameters in **Cell 4** (optional)
3. Run all cells sequentially
4. Results will be displayed inline and exported to `results/` directory

### Option 2: Julia REPL
```julia
include("Final_Project_Model.ipynb")  # Requires IJulia or notebook conversion
```
### Expected Runtime
- **16-week model:** ~3-5 minutes (single scenario)
- **52-week model:** ~15-30 minutes (single scenario)
- **Sensitivity analysis (8 scenarios):** ~25-40 minutes
 
## Output and Results
### Console Output
The model provides detailed progress information:
- Configuration summary
- Data loading statistics
- Model size (variables, constraints)
- Solver progress
- Optimal solution metrics

### Result Summaries (Cells 16-20)
1. **Data Center Performance:**
   - Curtailment frequency and energy
   - Economic value of flexibility
   - Top curtailment events
2. **Capacity Expansion:**
   - New generation capacity by technology and zone
   - Retirements
   - Storage additions
   - Transmission upgrades
3. **System Metrics:**
   - Reliability (loss of load probability)
   - Generation mix and renewable penetration
   - Cost breakdown (investment, O&M, fuel, NSE)

### Exported CSV Files (in `results/` directory)
| File | Description |
|------|-------------|
| `capacity_expansion.csv` | Generation capacity decisions by resource |
| `hourly_generation.csv` | Hourly dispatch for all generators |
| `datacenter_curtailment.csv` | Data center curtailment events and demands |
| `summary.csv` | High-level summary statistics |
| `sensitivity_analysis.csv` | Results across all strike price scenarios |

### Sensitivity Analysis
Tests strike prices from $100/MWh to $9,000/MWh to identify:
- Optimal strike price for system cost minimization
- Value of flexibility vs. firm load
- Curtailment patterns across price scenarios

## Project Structure
```
MAE573FP/
├── README.md                          # This file
├── Final_Project_Model.ipynb          # Main optimization model
├── results/                           # Output directory
│   ├── capacity_expansion.csv
│   ├── hourly_generation.csv
│   ├── datacenter_curtailment.csv
│   ├── summary.csv
│   └── sensitivity_analysis.csv
└── ercot_brownfield_expansion/        # Input data
    ├── readme.md                      # Data documentation
    ├── ercot_3_zone_map.png           # System map
    ├── 16_weeks/                      # Representative period data
    │   ├── Generators_data.csv
    │   ├── Generators_variability.csv
    │   ├── Load_data.csv
    │   ├── Fuels_data.csv
    │   └── Network.csv
    └── 52_weeks/                      # Full year data (optional)

```
--- 

## Replication Instructions
### Exact Replication of Published Results
1. Ensure all input data files are unchanged from the repository
2. Set parameters exactly as shown in Cell 4 (baseline configuration above)
3. Set `Random.seed!(42)` for reproducibility (already included in Cell 2)
4. Run all cells in order without modifications
5. Compare outputs to those in the `results/` directory

### Testing Alternative Scenarios
Modify the following parameters in Cell 4:
- **Data center location:** Change `DATA_CENTER_ZONE` (1, 2, or 3)
- **Strike price:** Adjust `STRIKE_PRICE` ($100 to $9,000/MWh)
- **Time horizon:** Set `USE_FULL_YEAR = true` for full 52-week analysis
- **Model features:** Toggle `ENABLE_STORAGE`, `ENABLE_RAMPING`, `ENABLE_TRANSMISSION_EXPANSION`

### Sensitivity Analysis
- Set `RUN_SENSITIVITY_ANALYSIS = true` to run all strike price scenarios
- Results will be saved to `results/sensitivity_analysis.csv`
- Warning: Runtime increases to ~25-40 minutes
---

## Troubleshooting
### Common Issues
1. **Out of Memory Error**
   - Reduce time horizon: Set `USE_FULL_YEAR = false`
   - Disable ramping: Set `ENABLE_RAMPING = false`
   - Increase system RAM or use high-memory compute environment

2. **Solver Time Limit Exceeded**
   - Increase timeout: `set_optimizer_attribute(Expansion_Model, "time_limit", 1200.0)`
   - Use fewer threads: `set_optimizer_attribute(Expansion_Model, "threads", 2)`
   - Simplify model: Disable storage or transmission expansion

3. **Package Installation Errors**
   - Update package registry: `Pkg.update()`
   - Check Julia version: `versioninfo()`
   - Install specific versions if needed: `Pkg.add(name="JuMP", version="1.0")`
---

**Last Updated:** December 2025

**Model Version:** 1.0

**Julia Version:** 1.6+