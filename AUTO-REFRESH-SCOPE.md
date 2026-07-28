# Hall of Fame auto-refresh scope

**Status:** Future pipeline, not built. The current `hall-data.json` remains a
dated snapshot until the live reputation source and publication workflow are
explicitly approved.

## Authority

- Resolve current Marketing scope, ownership, identifiers, and design-system
  routing through `AGENTS.md`.
- Resolve score definitions and the live reputation source from the owning
  Product or Signals repository. Historical handover folders are evidence, not
  a current integration contract.
- Keep Hall implementation and publication changes in this standalone
  repository. No parent-repository pointer update is part of publication.

## Proposed pipeline

1. Pull the current caller scores and roster from the approved live source.
2. Build and validate `hall-data.json` against the repository schema.
3. Regenerate share assets with `generate-og-card.py`.
4. Run repository checks, review the diff, and publish through the Hall
   repository's approved GitHub Pages workflow.

## Decisions required before implementation

- Live source endpoint, schema, and owning team.
- Refresh cadence.
- Tier-band authority.
- New-avatar policy.
- Runtime and credential owner.
- Failure alert and rollback procedure.

Do not create automation from this scope until those decisions exist in their
owning systems. The manual path remains: edit the snapshot, regenerate assets,
validate, review, and publish with explicit authorization.
