# Own the extension and use Pi model defaults

## Requirements

Bundled agents currently select `openrouter/z-ai/glm-5.3` even when the user has no credentials for that provider. The fix should work without agent instructions, machine-specific pins, or project-local copies of bundled profiles. Owning the extension also enables future availability notifications and other quality-of-life improvements.

## Final Decision

Maintain `LokiLin14/pi-interactive-subagents` as our fork. Fix the initial model-selection problem in the bundled profiles rather than in Pi Developer's APPEND_SYSTEM.md or local profile overrides.

The first proposed implementation removes bundled model pins and delegates fresh child selection to Pi's normal startup defaults/scopes. This means the saved user/project model, not necessarily the parent's currently selected model. Explicit model overrides and custom profile models remain supported.

Availability checks and clearer failure notifications are separate future work, not part of this initial fix. Pi Developer will adopt a tested, explicitly selected revision of this fork in a separate change.

## Alternatives considered

- Project-local profiles without model fields: technically feasible, but duplicates package-owned roles in every consuming project; the user rejected this approach in favor of owning the extension.
- Hardcoded accessible model: useful as a temporary local workaround, but not portable.
- Prompt instructions to forward the parent model: relies on agent compliance and does not provide the desired transparent behavior.
- Extension-level current-parent inheritance: possible future policy, but differs from the agreed saved-default preference and requires additional implementation.

## Related work

- [Task](../tasks/00-portable-model-defaults.md)
- [First PR plan](../plans/00-remove-bundled-model-pins.md)
