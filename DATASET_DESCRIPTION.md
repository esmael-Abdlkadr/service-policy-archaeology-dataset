# Service Policy Archaeology Dataset

## Overview

This is an original, fully synthetic dataset of 600 maintenance sites whose servicing policies are hidden. Each site has its own usage profile and its own set of maintenance tasks, and each task is governed by a rule drawn from a small typed menu: a calendar interval, a usage threshold, or a hybrid that fires on whichever comes first. The dataset contains no real operational records, no customer data and no third-party material; every row was produced by the generator described below.

The labels are the governing policy of each task: its mechanism, its parameters, and whether a service was due on each of a set of held-out candidate days.

## Release At A Glance

- Raw files: 11
- Sites (cases): 600
- Fleets (grouping keys): 30, 20 sites each
- Maintenance tasks per site: 8 to 15
- Observation window: 160 weeks, 1,120 days
- Usage readings: one cumulative hours figure per week per site, 96,000 rows in total
- Completed services: at least 5 per task, about 10,000 rows per 100 sites
- Candidate days per task: 12, half real due days and half decoys
- Mechanism menu: calendar, usage, whichever_first
- Calendar interval menu, days: 30, 45, 60, 90, 120, 180, 365
- Usage threshold menu, hours: 100, 250, 500, 750, 1000, 1500, 2000
- Prepared split: 24 fleets / 480 training sites, 6 fleets / 120 test sites
- Data origin: creator-generated synthetic data, version 1

## Raw File Structure

The uploaded ZIP is flat and contains exactly these eleven files at its root:

- `sites.csv`: one record per site: `case_id`, `family_id`, `n_tasks`.
- `usage.csv`: one record per site-week: `case_id`, `week`, `cumulative_hours`.
- `events.csv`: one record per completed service: `case_id`, `task_id`, `day`.
- `candidates.csv`: one record per candidate day: `case_id`, `task_id`, `candidate_day`.
- `labels.csv`: one creator-side record per site: `case_id` and `labels_json`, holding each task's mechanism, interval, threshold and due flags; used by `prepare.py` and never copied into public prepared data.
- `source_metadata.json`: provenance, scale, menus, seed policy and licence metadata.
- `LICENSE`: CC BY 4.0 notice and licence URL.
- `ATTRIBUTION.txt`: attribution text.
- `DATASET_CARD.md`: short scope and safety summary.
- `DATASET_DESCRIPTION.md`: this document.
- `PACKAGE_MANIFEST.sha256`: SHA-256 checksum of every other raw file.

## Columns

### sites.csv

- `case_id` (string): opaque site identifier, a keyed hash, for example `site_3f9c1a7b2e`.
- `family_id` (string): opaque fleet key, used only to build fleet-held-out splits.
- `n_tasks` (integer): number of maintenance tasks at the site.

### usage.csv

- `case_id` (string): joins to `sites.csv`.
- `week` (integer): 0 to 159.
- `cumulative_hours` (number): the site's usage counter at the end of that week, never decreasing.

### events.csv

- `case_id` (string) and `task_id` (string): the site and the maintenance task.
- `day` (integer): the day the service was completed, 0 to 1119.

### candidates.csv

- `case_id` (string) and `task_id` (string): as above.
- `candidate_day` (integer): a day to be judged as a real due day or a decoy.

### labels.csv

- `case_id` (string): joins to the other tables.
- `labels_json` (JSON object string): per task, `mechanism` (one of `calendar`, `usage`, `whichever_first`), `interval_days` (from the calendar menu, or null), `usage_threshold` (from the usage menu, or null), and `due` (a 0 or 1 flag per candidate day, in candidate order).

## How The Data Is Generated

Each site draws a usage profile: a base weekly rate, an annual seasonal swing of 0 to 75 percent, occasional idle spells, and week-to-week noise. The swing is the design point of the dataset. It makes a usage-driven task's gaps expand and contract in days while a calendar-driven task's gaps stay put, which is what makes the two mechanisms separable at all. About a fifth of sites are drawn with no swing, and on those the mechanisms are genuinely indistinguishable.

Each task draws a mechanism from the menu and its parameters. A hybrid's threshold is chosen near what the site actually accumulates over its interval, so that both components fire in different parts of the window, and a hybrid is only kept when each component fires at least 30 percent of the time. A task is only kept when it completes at least five services inside the window. This is deliberate: a rule whose second component never binds, or which barely appears in the log, would be unrecoverable in principle rather than merely hard.

Services are then simulated forward. The clock restarts from the last completion, so lateness accumulates as drift. Each completion lands 0 to 14 days after the due date depending on the site, a site-specific share of due services is skipped without a record while still restarting the clock, and visits are batched within a site-specific window so that a task can be pulled forward onto a day another task was serviced. Candidate days are then drawn per task, half from real due days and half from days comfortably clear of any due date.

All randomness, every `case_id`, `family_id` and `task_id` derive by HMAC-SHA256 from a 256-bit secret held by the creator. The public generator requires that secret and refuses to run without it, and the secret appears in no released file, in the source repository, or in its history. Sites are written in hashed-id order, so file order carries no generator index.

## Prepared Outputs

`prepare.py` writes a flat public directory plus a private answer directory. Every prepared file is keyed by `case_id` with one row per site.

- public `train.csv` (480 rows): `case_id`, `family_id`, `n_tasks`, `usage_json`, `events_json`, `candidates_json`.
- public `test.csv` (120 rows): the same columns for the held-out sites.
- public `train_labels.csv` (480 rows): `case_id` and `prediction_json`, the true mechanism, parameters and due flags of every training task, in the submission format.
- public `sample_submission.csv` (120 rows): `case_id` and `prediction_json`, structurally valid and deliberately empty of opinion.
- private `answers.csv` (120 rows): the same for the test sites, plus a reserved `__case_ids__` field listing them. It has the same columns as `sample_submission.csv`, and the grader ignores reserved fields, so the answer key is itself a perfect submission.

The public directory also holds `LICENSE`; no other raw document is copied into it, so nothing a solver receives names the dataset or its author.

## Characteristics

- Fleets group sites that share a usage profile family, and no fleet appears in both prepared splits.
- Mechanism mix is roughly 44 percent calendar, 36 percent usage, 21 percent hybrid.
- Identifiability is graded rather than uniform: the same solver recovers mechanisms far better on sites whose usage rate swings than on sites that run flat, which is why the mechanism target is scored as a calibrated belief.
- No column in any released file is ever empty, constant, or a token that a data profiler reads as a number.

## Known Limitations

- **Fully synthetic; no real-world validity.** The mechanisms, menus and noise model are design choices, not measurements of any real maintenance operation. Methods that work here need not transfer to real work-order data.
- **A finite, released menu.** Intervals and thresholds are drawn from published menus, so the parameter task is selection from a known set rather than open-ended estimation.
- **Uniform site shape.** Every site has the same observation window and the same number of candidate days per task.
- **Some tasks are unidentifiable by construction.** On a constant-rate site a calendar rule and a usage rule produce the same log. This is intentional and is why the scoring rules are proper, but it caps what any solver can reach.
- **Reproducibility is restricted by design.** The generator is public, but the released data can be regenerated only with the withheld secret. Anyone auditing it can run it with their own secret to obtain a statistically equivalent dataset, not this one.

## Provenance And License

All records are generated by an original deterministic synthetic generator written for this dataset. The package is released under Creative Commons Attribution 4.0 International (CC BY 4.0). Attribution: Esmael Abdlkadr, Service Policy Archaeology Dataset (2026).
