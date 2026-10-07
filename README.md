# Building Energy Optimization — Doctoral Thesis Code and Data

Simulation models, optimization notebooks and result files of a doctoral thesis on the multi-objective optimization of the thermal energy performance of residential buildings in a subtropical climate (Passo Fundo, Rio Grande do Sul, Brazil).

The work couples **EnergyPlus** models with the **NSGA-II** genetic algorithm through the **BESOS** Python library, minimizing **cooling demand** and **heating demand** simultaneously. It is organized in the three phases of the thesis:

| Phase | Name in the thesis | Decision variables |
|---|---|---|
| I | **Isolated scenario** | Building orientation |
| II | **Surroundings** | Building orientation, with the surrounding buildings modelled |
| III | **Envelope optimization integrated into surroundings** | Nine envelope and natural-ventilation variables |

Two building typologies are modelled in every phase: the **H Building** (H-shaped plan) and the **Linear Building**.

## Repository structure

```
.
├── phase-1-isolated-scenario/
│   ├── h-building/        models/  notebooks/  results/
│   └── linear-building/   models/  notebooks/  results/
├── phase-2-surroundings/
│   ├── h-building/        models/  notebooks/  results/
│   └── linear-building/   models/  notebooks/  results/
├── phase-3-envelope-optimization/
│   ├── h-building/        models/  notebooks/  results/
│   └── linear-building/   models/  notebooks/  results/
├── weather/               EPW weather file (Passo Fundo, RS)
├── figures/               Cooling and heating demand by floor (Phases I and II)
├── docs/file-mapping.csv  Original file names -> paths in this repository
├── requirements.txt
├── LICENSE                MIT (code and notebooks)
└── LICENSE-DATA.md        CC BY 4.0 (models, results, figures)
```

Inside each `building/` folder:

- `models/` — EnergyPlus input files (`.idf`).
- `notebooks/` — Jupyter notebooks that load the model, define the variables and objectives, run NSGA-II and save the results.
- `results/` — CSV files written by the notebooks.

## Phases and models

**Phase I — Isolated scenario.** Each building is simulated alone, without surroundings. The only decision variable is the building orientation (`North Axis` of the EnergyPlus `Building` object, 0–359°).

**Phase II — Surroundings.** Same variable and objectives, with the surrounding buildings included in the model, to quantify how the context changes the optimal orientation and the demand.

**Phase III — Envelope optimization integrated into surroundings.** The surroundings of Phase II are kept and nine envelope and ventilation variables are optimized. For each building there are four model variants:

| Variant | Description |
|---|---|
| `rb` | Baseline model |
| `wwr15`, `wwr20`, `wwr25` | Window-to-wall ratio (WWR) scenarios of 15 %, 20 % and 25 % |

Decision variables of Phase III (as named in the notebooks and result files):

| Variable | EnergyPlus object | Field | Range |
|---|---|---|---|
| `IT - North / South / East / West Ext. Walls` | `MATERIAL` `Rockwool_north/south/east/west` | Thickness | 0.0001–0.15 m |
| `IT - Ext. Roof` | `MATERIAL` `Rockwool_roof` | Thickness | 0.0001–0.15 m |
| `SA - Ext. Walls` | `MATERIAL` `Ext mortar` | Solar absorptance | 0.2–0.8 |
| `SA - Ext. Roof` | `MATERIAL` `Fibercement tile` | Solar absorptance | 0.2–0.8 |
| `Glazing - Thickness` | `WINDOWMATERIAL:GLAZING` `Clear 3mm` | Thickness | 0.003–0.01 m |
| `AFN - Setpoint` | `SCHEDULE:COMPACT` `VN_19` | Natural-ventilation (AirflowNetwork) control setpoint | 18–25 °C |

`IT` = insulation thickness, `SA` = solar absorptance, `AFN` = AirflowNetwork.

## Objectives and optimization setup

- **Objectives (both minimized):** annual cooling demand (`DistrictCooling:HVAC` meter) and annual heating demand (`DistrictHeating:HVAC` meter), from the ideal-loads HVAC representation.
- **Units:** the notebooks convert EnergyPlus output from joules to **kWh/m²·year**, dividing by the floor area set in each notebook (H Building: 876.15 m²; Linear Building: 884 m²).
- **Algorithm:** NSGA-II (BESOS / Platypus). Population size and number of evaluations are set in each notebook.
- **Weather:** `weather/BRA_RS_Passo.Fundo.869630_TMYx.2007-2021.epw` (TMYx 2007–2021, WMO 869630, Passo Fundo, RS). It is a third-party file from the Climate.OneBuilding.Org TMYx collection; please check the source's terms before redistributing it.

## Result files

Each `results/*.csv` has one row per solution returned by NSGA-II, with these columns:

- the decision variables (`Orientation` in Phases I–II; the nine variables above in Phase III);
- `Cooling demand (kWh/m²)` and `Heating demand (kWh/m²)`;
- `violation` (constraint violation, 0 when none) and `pareto-optimal` (`True` for non-dominated solutions).

## Reproducing the runs

1. Install **EnergyPlus**. The models carry the version identifier 9.0; if you use a newer EnergyPlus, update the `.idf` files with the EnergyPlus version-transition tool first.
2. Create a Python environment (the notebooks record Python 3.9) and install the dependencies:

   ```bash
   pip install -r requirements.txt
   ```

3. Open a notebook and run it **from its own folder** (paths are relative to `notebooks/`):

   ```bash
   cd phase-3-envelope-optimization/h-building/notebooks
   jupyter lab
   ```

   The notebook reads `../models/*.idf` and `../../../weather/*.epw`, and writes the CSV and the Pareto plot to `../results/`. Long runs benefit from a parallel Dask client; see the BESOS documentation.

The notebooks in the repository keep their saved outputs from the original runs. Exact package versions were not recorded at the time; `requirements.txt` is therefore unpinned.

## Notes on file organization

- File names were standardized (no spaces, dates or `%`). `docs/file-mapping.csv` lists the original name of every file.
- Notebook text and comments are in English; the code logic is unchanged. Only the file paths were updated.
- One Phase III results file (`linear-building`, `wwr20`) was re-exported with a comma separator and decimal point to match the others; the values are the same.
- Phase I and II figures show cooling and heating demand by floor (F1–F5).

## License

Code and notebooks: MIT (see `LICENSE`). Models, results and figures: CC BY 4.0 (see `LICENSE-DATA.md`). The weather file is third-party material and is not covered by these licenses.

## How to cite

If you use this material, please cite the doctoral thesis (Letiane Benincá, PhD, joint supervision UFRGS and UPC Barcelona, 2024). A DOI for this repository and a full reference will be added here.

## Author

Letiane Benincá — GitHub: [LetianeBeninca](https://github.com/LetianeBeninca)
