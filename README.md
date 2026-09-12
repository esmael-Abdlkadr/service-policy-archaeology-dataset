# Service Policy Archaeology Dataset

An original synthetic benchmark of 600 maintenance sites whose servicing policies are hidden. Each site provides a weekly usage record and a log of completed services; the labels are the governing rule of every maintenance task: whether it is driven by a calendar interval, by a usage threshold, or by whichever falls due first, the parameters of that rule, and the schedule it implies.

- Licence: CC BY 4.0. Attribution: Esmael Abdlkadr, Service Policy Archaeology Dataset (2026).
- Contents of this repository: `DATASET_DESCRIPTION.md`, the full dataset card, identical to the copy inside the release archive, and `LICENSE`.
- The release archive itself is distributed on the challenge platform, not here.

## Reproducibility and answer safety

Every random draw, identifier and timestamp in the release is derived by HMAC-SHA256 from a 256-bit secret held by the creator and never published. The generator cannot be run by anyone else to reproduce the released sites or their answer keys, and no seed, index or identifier scheme in the release is enumerable back to them. Running an equivalent generator with a different secret yields a statistically equivalent dataset, not this one.

The construction procedure is documented in full in `DATASET_DESCRIPTION.md`: the mechanism and parameter menus, the usage model, the noise model covering technician delay, batched visits and skipped services, the rules that keep every task recoverable, and the split. Reviewers who need the construction code to audit those claims can request it.

The dataset contains no real operational records, no customer data and no third-party material. Every row was produced by the generator.
