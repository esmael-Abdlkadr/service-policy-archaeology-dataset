# Service Policy Archaeology Dataset: Coupled Maintenance Rules Behind Sparse Meter Logs

An original synthetic benchmark of 5,000 maintenance sites, each run by its own operator, so every site is an independent unit. Every site keeps a work log of completed services, carried out on technician route visits every 7 or 14 days, and irregular, partly missing readings of two or three usage meters, some of which are replaced and restart from zero. Every maintenance task follows a hidden servicing policy: a calendar interval, a usage threshold on one of the site's meters, or whichever of the two comes first. Some tasks also have a hidden parent, and are serviced on the parent's visit when they are far enough through their own cycle. The labels are the complete policy of every task: mechanism, counted meter, interval, threshold and parent.

- Licence: CC BY 4.0. Attribution: Esmael Abdlkadr, Service Policy Archaeology Dataset (2026).
- Contents of this repository: `DATASET_DESCRIPTION.md`, the full dataset card, identical to the copy inside the release archive, and `LICENSE`.
- The release archive itself is distributed on the challenge platform, not here.

## Reproducibility and answer safety

Every random draw, every identifier and the component-type tendencies are derived by HMAC-SHA256 from a 256-bit secret held by the creator and never published. No seed, index or identifier scheme in the release can be enumerated back to the generator state. The generator code is withheld together with the secret.

The construction procedure is documented in full in `DATASET_DESCRIPTION.md`: the usage and meter-reading model, the policy menus and component-type tendencies, the parent coupling rule, the operational noise (late completion, batched visits, silent skips), the rules that keep every label recoverable, and the site-level split (4,000 training sites, 1,000 test sites).

The dataset contains no real operational records, no customer data and no third-party material.
