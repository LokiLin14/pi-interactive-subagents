# Make bundled subagents use portable model defaults

## Short Description

Implement the [fork/model-default decision](../decisions/00-use-pi-model-defaults.md) so a fresh bundled agent does not force an unrelated provider. The [first PR plan](../plans/00-remove-bundled-model-pins.md) covers the package change; consumer adoption belongs in Pi Developer.

## Queue

- [x] Remove bundled model pins while preserving roles, permissions, and lifecycle behavior.
- [x] Cover model-less command construction and explicit model behavior with regression tests against real extension helpers.
- [x] Document default selection, override precedence, thinking behavior, and resume limitations accurately.
- [x] Record tests, an explicitly authorized runtime smoke check, and independent feature/style review results in the plan.

## Out of scope

- Availability preflight checks, failure notifications, retries, automatic model fallback, and other QoL features.
- Inheriting the parent's current model or changing Pi's startup model-selection algorithm.
- Changing tool isolation, permission boundaries, or authentication configuration.
- Consumer dependency updates, package publication, commits, pushes, and PR actions without separate authorization.
