# Maintenance-scheduling environment parameters

This repository contains a manuscript-aligned parameter transcription for the maintenance-scheduling instances. It contains JSON parameter data and field documentation. It does not contain training settings, learning algorithms, solver implementations, checkpoints, experimental results, or manuscript files.

## Files

| File | Contents |
| --- | --- |
| `parameters/common.json` | Time constants, maintenance actions, restoration factors, environment execution rules, and violation-penalty settings. |
| `parameters/instances/12_machines.json` | Machine physical parameters, topology, action-specific part requirements, required skills, processing times, and two-worker profiles. |
| `parameters/instances/24_machines.json` | The corresponding 24-machine instance with eight stages and four workers. |
| `parameters/instances/48_machines.json` | The corresponding 48-machine instance with sixteen stages and eight workers. |
| `parameters/resource_grid_24.json` | Three inventory-supply factors crossed with three workforce configurations. |
| `parameters/supplementary_conditions.json` | Replenishment-interval, qualification-overlap, fixed-tight-inventory workforce, and execution-rule variants. |

## Conventions

Machine IDs and stage IDs start at 1. Worker IDs and skill IDs start at 0. Every part vector uses the order `S1, S2, S3`. Qualification vectors and efficiency vectors use skill order `0, 1, 2`. A qualification entry of 1 means that the worker is qualified. A null efficiency entry denotes an unqualified worker/skill pair.

The action order is NM, PM1, PM2, PM3, CM. Their restoration levels are 0, 0.4, 0.6, 0.8, and 0.98; their remaining-age factors are 1, 0.6, 0.4, 0.2, and 0.02. CM is mandatory for failed machines and can also be selected for healthy machines. PM1 and PM2 consume no parts. PM3 consumption depends on the machine; a zero vector for an individual machine is allowed.

Time values use the model time unit. An episode contains 20 decision windows, each with maintenance-window length 1 and production-period length 20. The baseline delivery interval is five windows. Initial-stock ranges are inclusive discrete uniform integer distributions. The files specify the interval without adding an unverified first-delivery offset.

`cm_time` is the nominal corrective-maintenance duration. A preventive duration equals `cm_time` multiplied by its restoration level. Worker efficiency is a **duration multiplier**: worker-dependent duration equals nominal duration times the multiplier, so a smaller positive value corresponds to faster work.

The machine fields `shape` and `scale` are the Weibull parameters. `lifetime`, `bias`, and `A_condition` preserve the corresponding instance-table fields. `A_cap` is the capacity field used for the single product in these instances. Field names are retained to make comparison with the instance tables possible.

## Resource grid

| Supply factor | Initial S1 / S2 / S3 stock intervals | Units per delivery |
| --- | --- | --- |
| 1.00 | [29,45] / [23,35] / [11,18] | 31 / 19 / 7 |
| 0.75 | [21,33] / [17,26] / [8,13] | 23 / 14 / 5 |
| 0.50 | [14,22] / [11,17] / [5,9] | 15 / 9 / 3 |

| Workers | Retained baseline worker IDs | Qualified workers per skill |
| --- | --- | --- |
| 4 | 0, 1, 2, 3 | 2, 2, 2 |
| 3 | 0, 2, 3 | 1, 2, 2 |
| 2 | 0, 2 | 1, 2, 1 |

The nine grid entries inherit the machine data, action requirements, and retained worker profiles of the 24-machine instance. Worker IDs remain the original baseline IDs when a subset is selected. Inventory factors multiply each baseline bound and delivery quantity, followed by rounding down. Inventory-capacity values are not specified in this release because the current manuscript does not specify that constraint.

## Provenance and scope

Core constants and the resource grid follow the current manuscript's experimental setup and the resource-configuration table in the accompanying author response. Machine physical parameters, per-machine part requirements, required skills, and baseline worker profiles come from the existing 12-, 24-, and 48-machine instance tables. The sparse per-machine part requirements are expanded into explicit vectors, preserving the manuscript's zero consumption at PM1 and PM2. Supplementary qualification matrices follow the accompanying resource-settings record.

This is a field-level parameter transcription, not an unchanged export of an archived experimental run. Historical configuration files contain conflicting parts-use, monetary, and downtime settings. The operating-time expression in `common.json` follows the manuscript. Monetary coefficients such as product price, labor charges, part prices, setup charges, emergency procurement rates, and spillover rates are not assigned in this version. Initial machine-state sampling, initial delivery phase, alternative-overlap worker efficiencies, and scale-specific inventory settings are also outside the verified fields in this release. Consequently, these files alone do not specify a complete executable simulation or establish reproduction of reported costs.

Algorithmic settings such as PPO/CPO options, BC coefficients, priority-score weights, training/evaluation seeds, training budgets, and solver budgets are excluded. The environment violation-penalty coefficients remain included because they define the environment's reward, and actual economic cost excludes those discretionary-violation charges.
