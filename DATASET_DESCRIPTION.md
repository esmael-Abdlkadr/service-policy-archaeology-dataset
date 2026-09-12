# Service Policy Archaeology Dataset v2: Coupled Maintenance Rules Behind Sparse Meter Logs

## Overview

This is an original, fully synthetic dataset of 1,300 maintenance sites run by 65 operators. Every site keeps a work log of completed services and irregular readings of two or three usage meters, and every maintenance task on the site follows a hidden servicing policy. The dataset contains no real operational records, no customer data and no third-party material; every row was produced by the generation procedure described below.

The labels are the governing policy of each task: which mechanism drives it (a calendar interval, a usage threshold on one meter, or whichever of the two comes first), which meter it counts, the interval and threshold values, and which other task on the site, if any, is its parent. A child task is serviced on its parent's visit when it is far enough through its own cycle.

## Release At A Glance

- Raw files: 12
- Sites (cases): 1,300
- Operators (grouping keys): 65, 20 sites each
- Maintenance tasks: 15,057, 8 to 15 per site
- Meters: 3,236, 2 or 3 per site, of kind hours, starts or cycles
- Meter readings: 121,975, between 11 and 84 per meter
- Completed services: 251,493, between 5 and 60 per task
- Observation window: days 1 to 1,120
- Component types: 12
- Mechanism mix: 52.8 percent calendar, 34.4 percent usage, 12.7 percent whichever_first
- Tasks with a parent: 26.4 percent
- Interval menu, days: 30, 45, 60, 90, 120, 180
- Threshold menu, meter units: 100, 250, 500, 750, 1000, 1500, 2000, 3000
- Prepared split: 53 operators / 1,060 training sites, 12 operators / 240 test sites
- Data origin: creator-generated synthetic data, version 2

## Raw File Structure

The uploaded ZIP is flat and contains exactly these twelve files at its root:

- `sites.csv`: one record per site: `case_id`, `family_id`, `n_tasks`, `n_meters`.
- `meters.csv`: one record per meter: `case_id`, `meter_id`, `kind`.
- `tasks.csv`: one record per maintenance task: `case_id`, `task_id`, `component_type`.
- `readings.csv`: one record per meter reading: `case_id`, `meter_id`, `day`, `value`.
- `events.csv`: one record per completed service: `case_id`, `task_id`, `day`.
- `labels.csv`: one creator-side record per site: `case_id`, `labels_json`; used by `prepare.py` and never copied into public prepared data.
- `source_metadata.json`: provenance, scale, menus, generation policy and licence metadata.
- `LICENSE`: CC BY 4.0 notice and licence URL.
- `ATTRIBUTION.txt`: attribution text.
- `DATASET_CARD.md`: short scope and safety summary.
- `DATASET_DESCRIPTION.md`: this document.
- `PACKAGE_MANIFEST.sha256`: SHA-256 checksum of every other raw file.

## Columns

### sites.csv

- `case_id` (string): opaque site identifier, a keyed hash, for example `site_3f9c1a7b2e`.
- `family_id` (string): opaque operator key, for example `op_3e422bdb`. Sites of one operator share its working habits: lateness, skipping, batching and meter-reading frequency.
- `n_tasks` (integer): number of maintenance tasks at the site, 8 to 15.
- `n_meters` (integer): number of meters at the site, 2 or 3.

### meters.csv

- `case_id` (string): joins to `sites.csv`.
- `meter_id` (string): opaque meter identifier, unique within the release.
- `kind` (string): `hours`, `starts` or `cycles`. A site never has two meters of the same kind.

### tasks.csv

- `case_id` (string): joins to `sites.csv`.
- `task_id` (string): opaque task identifier, unique within its site; always use it together with `case_id`.
- `component_type` (string): opaque component type, one of 12, for example `ct_ceaab5`. Component types have typical but never decisive policies.

### readings.csv

- `case_id` (string) and `meter_id` (string): the site and the meter.
- `day` (integer): day of the reading, 1 to 1,120.
- `value` (integer): the counter shown on that day. Counters normally only rise; a counter that was replaced restarts from zero, so its readings drop once at the replacement. 34.7 percent of meters show such a drop.

### events.csv

- `case_id` (string) and `task_id` (string): the site and the task.
- `day` (integer): the day the service was completed, 1 to 1,120.

### labels.csv

- `case_id` (string): joins to the other tables.
- `labels_json` (JSON object string): per task, `mechanism` (`calendar`, `usage` or `whichever_first`); `meter` (the counted meter's id, or `no_meter` for a calendar rule); `interval_days` (from the interval menu, or 0 when the rule has no calendar part); `usage_threshold` (from the threshold menu, or 0 when the rule counts no usage); `parent` (another task id of the same site, or `no_parent`).

## How The Data Is Generated

**Usage.** Each site has a shared load profile: an annual seasonal swing of 0 to 60 percent, week-to-week variation, and one to eight idle spells of one to six weeks during which usage almost stops. Each meter has its own base daily rate for its kind and mixes the shared load with an independent profile of its own, so meters on one site are correlated to different degrees. About a third of meters are replaced once, and their counters restart from zero.

**Readings.** Each operator reads meters on a nominal cycle of 14, 28, 42 or 63 days. Each gap varies between half and one and a half times that cycle, and about one reading in ten is missing. Reading days are drawn independently of service days; 12.4 percent of readings happen to fall on a day with a service.

**Policies.** Each task draws a component type, and the type's tendencies weight the draws of mechanism, meter kind, interval and threshold. The rules are:
- `calendar`: service falls due the given number of days after the last service.
- `usage`: service falls due once the named meter has advanced by the threshold since the last service.
- `whichever_first`: service falls due at the earlier of the two. Its threshold is chosen near what the meter accumulates over the interval, and a hybrid is kept only when each component triggers between 30 and 70 percent of its services.

About three tasks in ten are given a parent among the site's other tasks.

**Operations.** Sites are simulated day by day from the true meter trajectories. The clock of a task restarts at every completion.
- A due service is completed 0 to 14 days late, depending on the operator.
- 0 to 15 percent of due services are skipped silently, still restarting the clock.
- On a visit, tasks due within the operator's batching window of 0, 2 or 4 days are pulled forward half of the time.
- When a parent is serviced, a child that is at least 60 percent through its cycle is serviced on the same day.
- A task is kept only with 5 to 60 completions.
- A parent link that never pulled its child forward during the window leaves no trace in the data and is recorded as `no_parent`.

**Keys and identifiers.** All randomness, every identifier and the component-type tendencies derive by HMAC-SHA256 from a 256-bit secret held by the creator. The secret and the generator code are withheld, and the secret appears in no released file. Sites are written in hashed-id order, and tasks and meters in hashed-id order within a site, so no ordering carries generator information.

## Prepared Outputs

`prepare.py` writes a flat public directory plus a private answer directory. Every prepared file is keyed by `case_id` with one row per site.

- public `train.csv` (1,060 rows): `case_id`, `family_id`, `n_tasks`, `meters_json`, `tasks_json`, `readings_json`, `events_json`. The JSON columns hold the site's rows from `meters.csv`, `tasks.csv`, `readings.csv` (as `[meter_id, day, value]`) and `events.csv` (as `[task_id, day]`).
- public `test.csv` (240 rows): the same columns for the held-out sites.
- public `train_labels.csv` (1,060 rows): `case_id` and `prediction_json`, the true policy of every training task in the submission format, `{"tasks": {task_id: {...}}}`.
- public `sample_submission.csv` (240 rows): `case_id` and `prediction_json`, structurally valid with no opinion on any task.
- private `answers.csv` (240 rows): the same for the test sites, plus reserved fields listing each site's meters and the complete set of test case IDs. It has the same columns as `sample_submission.csv`, and the grader ignores reserved fields, so the answer key is itself a perfect submission.

The public directory also holds `LICENSE`; no other raw document is copied into it, so nothing a solver receives names the dataset or its author. The preparation self-check verifies that the key agrees with the public inputs: every labelled task, meter and parent exists on its site, and every rule's fields are consistent with its mechanism.

## Characteristics

- Operators never appear in both prepared splits, so their habits must be inferred rather than memorised.
- Identifiability is uneven by construction. A flat usage profile makes calendar and usage rules hard to separate, correlated meters make the counted meter hard to name, and a parent that pulled its child only once or twice leaves thin evidence. This is why every target is a calibrated belief rather than a single answer.
- No column in any released file is ever empty, constant, or a token that a data profiler reads as a missing or infinite number.

## Known Limitations

- **Fully synthetic; no real-world validity.** The mechanisms, menus, coupling rule and noise model are design choices, not measurements of any real maintenance operation.
- **Finite, released menus.** Intervals and thresholds are selected from published menus rather than estimated on a continuous scale.
- **One kind of coupling.** Parents pull children forward by a single opportunistic rule. A task has at most one parent, and chains of parents are possible.
- **Fixed window.** Every site is observed over the same 1,120 days.
- **Restricted reproducibility.** The generation procedure is documented here, but the released data can be regenerated only with the withheld secret.

## Provenance And License

All records are produced by an original synthetic generation procedure written for this dataset. The package is released under Creative Commons Attribution 4.0 International (CC BY 4.0). Attribution: Esmael Abdlkadr, Service Policy Archaeology Dataset v2 (2026).
