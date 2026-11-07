# Multi-Machine Maintenance Scheduling Instances with Spare-Part and Workforce Constraints

This repository provides environment parameters accompanying the EASPO maintenance-scheduling study. The instances describe manufacturing systems with machine degradation, multiple maintenance levels, spare-part inventories, and workers with overlapping skills.

The files cover the 12-, 24-, and 48-machine systems, the 24-machine inventory–workforce resource grid, and supplementary operating conditions. All parameters are provided in JSON format.

## Repository contents

| File | Description |
| --- | --- |
| [common.json](parameters/common.json) | Time constants, maintenance actions, restoration factors, processing-time definitions, execution rules, and environment violation penalties. |
| [12_machines.json](parameters/instances/12_machines.json) | Physical machine parameters, topology, part requirements, required skills, and worker profiles for 12 machines and 2 workers. |
| [24_machines.json](parameters/instances/24_machines.json) | The corresponding parameters for 24 machines and 4 workers. |
| [48_machines.json](parameters/instances/48_machines.json) | The corresponding parameters for 48 machines and 8 workers. |
| [resource_grid_24.json](parameters/resource_grid_24.json) | Initial-stock distributions, replenishment quantities, and worker subsets for the nine resource configurations. |
| [supplementary_conditions.json](parameters/supplementary_conditions.json) | Replenishment-interval, skill-overlap, workforce-size, and execution-rule variants. |

## Baseline setting

The main case contains 24 machines arranged in eight stages, with **4, 2, 4, 4, 2, 2, 2, and 4 machines** per stage. Four workers cover three skills, and maintenance uses three spare-part types: S1, S2, and S3.

Each episode contains 20 decision windows. The maintenance window has length 1, the production period has length 20, and replenishment occurs at intervals of five windows. Time lengths use the model time unit.

| Action | Description | Restoration level | Remaining-age factor | Parts (S1, S2, S3) |
| --- | --- | --- | --- | --- |
| NM | No maintenance | 0 | 1 | (0, 0, 0) |
| PM1 | Preventive maintenance, level 1 | 0.4 | 0.6 | (1, 0, 0) |
| PM2 | Preventive maintenance, level 2 | 0.6 | 0.4 | (1, 1, 0) |
| PM3 | Preventive maintenance, level 3 | 0.8 | 0.2 | (2, 1, 1) |
| CM | Corrective replacement | 0.98 | 0.02 | (2, 2, 1) |

All PM levels and CM consume spare parts and qualified worker time. The part vectors in the table apply to every machine in the 12-, 24-, and 48-machine instances and are also listed in each instance file. CM is mandatory for failed machines and may also be selected for healthy machines. Mandatory corrective work is handled first; the default execution rule accepts discretionary work on an all-or-nothing basis.

## Inventory and workforce configurations

The resource study crosses the three inventory-supply settings below with each of the three workforce configurations, producing nine combinations. The baseline uses supply factor 1.00 and four workers.

| Supply factor | Initial S1 stock | Initial S2 stock | Initial S3 stock | Delivery quantities (S1, S2, S3) |
| --- | --- | --- | --- | --- |
| 1.00 | [29, 45] | [23, 35] | [11, 18] | (31, 19, 7) |
| 0.75 | [21, 33] | [17, 26] | [8, 13] | (23, 14, 5) |
| 0.50 | [14, 22] | [11, 17] | [5, 9] | (15, 9, 3) |

Initial stocks follow discrete uniform distributions over the inclusive integer intervals. Each supply factor scales the baseline stock bounds and delivery quantities, with componentwise rounding down.

| Workers | Retained worker IDs | Qualified-worker counts by skill |
| --- | --- | --- |
| 4 | 0, 1, 2, 3 | (2, 2, 2) |
| 3 | 0, 2, 3 | (1, 2, 2) |
| 2 | 0, 2 | (1, 2, 1) |

Retained workers keep their baseline qualifications and efficiencies. Their IDs remain unchanged when selecting a subset. Machine parameters, action requirements, and maintenance-window length remain fixed across the resource grid.

## Reading the files

From the repository root, load the common settings, machine instance, and resource configuration together:

```python
import json
from pathlib import Path

parameters = Path("parameters")
common = json.loads((parameters / "common.json").read_text())
instance = json.loads((parameters / "instances/24_machines.json").read_text())
resources = json.loads((parameters / "resource_grid_24.json").read_text())

print(instance["machine_count"])  # 24
print(resources["default"])      # {'supply_factor': 1, 'worker_count': 4}
```

Machine and stage IDs start at 1; worker and skill IDs start at 0. Part vectors follow the order **S1, S2, S3**, and skill vectors follow **0, 1, 2**. A qualification entry of 1 means qualified; a null efficiency entry denotes an unqualified worker–skill pair.

The machine fields `shape` and `scale` specify the Weibull parameters, and `A_cap` gives the capacity for the modeled product. The field `cm_time` gives the nominal CM duration. A PM duration equals `cm_time` multiplied by the restoration level. Worker-dependent duration then equals nominal duration multiplied by `efficiency_by_skill`; a smaller positive multiplier means faster processing.

For each production period, the time to first failure is sampled from the residual-life distribution conditional on the post-maintenance age. The same sample determines the failure indicator and the operating time `min(T_fail, tau_p)`.

## Publication scope

The published fields follow the manuscript's case setting, resource design, and supplementary configuration descriptions. This repository distributes environment parameter data; training configurations, algorithms, checkpoints, and experimental results are outside its scope.

The release does not supply monetary coefficients, inventory-capacity limits, initial machine-state sampling, the initial replenishment phase, efficiencies for the alternative skill-overlap condition, or inventory settings specific to the 12- and 48-machine systems. These files therefore provide a parameter reference rather than a complete simulator configuration.
