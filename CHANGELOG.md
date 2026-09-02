# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]
### Added
* `coveragemass.py`: implemented **Coverage by Duration** (`cov_Δt`, Definition 4.1),
  a time-based alternative to `coverage_mass` that estimates process coverage from
  the implied duration of activities present in the model but missing from the log,
  rather than from model transition weights.
  * `dur(sigma)` - duration of a trace from its first/last event timestamps.
  * `mi(activities, sigma)` - trace positions (excluding the first event) whose
    activity is in a given alphabet.
  * `sdur(activities, sigma, has_submodels)` - duration attributable to a
    (sub)model: summed across occurrences for composite/operator nodes,
    averaged across occurrences for a single leaf activity.
  * `coverage_by_duration(pt, log, skip_probs, total_dur=None)` - computes
    `cov_Δt` for any `ProcessTree` node `pt`, discounted by the same
    `P_skip` used elsewhere (`dv.skip_probs`). Accepts an optional
    precomputed `total_dur` to avoid re-summing the log's total duration
    when evaluating many nodes over the same log.
  * `log_to_traces(log)` - normalises a pm4py event log (pandas `DataFrame`
    or classic `EventLog`) into a list of ordered per-trace event lists, as
    expected by the functions above.
* `pvoid.py`: added `show_tree_coverage_by_duration(tree, dv, traces, total_dur=None)`,
  a per-node tree printer analogous to `show_tree_weights`, but printing each
  node's `coverage_by_duration` instead of its SLPN-derived `.weight`.

### Changed

* Extracted the shared skip-probability engine (process tree, alignment,
  execution, probability derivation) into its own library,
  [skip-alignments](https://github.com/adamburkegh/skip-alignments), which
  process-voids now depends on instead of vendoring copies of the same code.
* Switched to a `pyproject.toml`-based install (`pip install -e .`) as the
  documented way to install process-voids and its dependencies.
* `pvoid.py` `main()`: now builds `traces` via `log_to_traces(logx)` and a
  shared `total_dur`, then:
  * prints the tree via `show_tree_coverage_by_duration(...)` instead of
    `show_tree_weights(...)`,
  * reports `Coverage by Duration: {coverage_by_duration(pt, traces, dv.skip_probs, total_dur)}`
    instead of `Coverage: {coverage_mass(pt, dv.skip_probs)}`.
### Deprecated
- `coveragemass.py`: `coverage_mass(pt, skip_probs)` is commented out (not
  deleted). It remains available for reference / rollback but is no longer
  called anywhere in `pvoid.py`.

### Removed

* Removed the duplicated engine modules (`alignall.py`, `alignment.py`,
  `derivation.py`, `execution.py`, `probabilities.py`, `processtree.py`,
  `skips.py`) from this repo; the same code now lives only in
  `skip-alignments`.
  
### Unchanged
- `update_activity_weights`, `infer_operator_weights`, `transfer_pt_weights`,
  and `show_tree_weights` are untouched and still run; `.weight` is still
  populated on every tree node even though nothing currently prints it via
  the new coverage path. These can be removed later if the weight-based
  view is no longer needed.
### Verified
- Cross-checked `coverage_by_duration` leaf output against the raw
  `rtfm_fine_appeal_xes.gz` log by independently computing per-activity
  duration shares from the XES timestamps: all leaf values reproduce
  exactly via `cov = (1 - P_skip) * raw_duration_share`, and the root value
  equals `1 - P_skip(root)` since every real activity's duration share
  sums to 1.0 of total log duration (only `Tau` nodes are excluded from
  the alphabet).
  
## [0.3.1] - Adding BPMN Colour According to Skip Probabilities - #3

### Added

* Added a visualisation tool for process-voids metric. Colour BPMN activities according to their metric values. Can be rendered using `bpmn.io`. 
