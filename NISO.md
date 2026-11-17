# niso's taffy

This is DioxusLabs/taffy with niso's patches on top. niso uses it through its blitz fork.

**Upstream commit:** 4aca4f4933aa9cb7a3266050c788aad4c90288d2 on DioxusLabs/taffy `main`
("Resolve percentage widths against zero in the min-content contribution of replaced block children (#1401)").

## Patches

Each patch is one commit, with Chrome-generated fixtures in `test_fixtures/`.

- **A margin that collapses through a block's top moves the block, and its floats are checked from there** (`src/compute/block.rs`).
  A child's position counts the block's own unresolved top strut, and once that strut resolves, the content after it is placed against floats at the block's resolved position.
  Why: niso's TodoMVC footer. `.todoapp`'s 130px top margin escapes `body`, so the footer's floats were checked 130px too low, which narrowed the line of the third info paragraph.
  Fixtures: `float_after_margin_collapsed_through_parent` fails without the patch.
  `float_before_block_with_margin_collapsed_through_it` passes with and without it, and keeps it that way.
  Upstream PR #1345 fixes the first case but breaks the second, so it isn't used.

## Known upstream faults

These faults are in upstream at the commit above. The patches don't change them.

- **A float placed before a margin that collapses through its parent doesn't move down with the parent.**
  Example: a 400px block holds a block whose first child is a right float, 50 by 50, followed by a block with `margin-top: 100px` that holds a right float, 50 by 20.
  Chrome puts the parent at y 100, the first float at x 350, y 0 inside it, and the second float at x 300, beside the first.
  Taffy puts the second float at x 350, on top of the first, because the first float stays at y 0 in the float space.
  The patch doesn't cover this, for two reasons:
  - the float space can't move a placed float, because each float's segments hold its containing-block insets, which the placed box doesn't keep;
  - how far each float should move depends on which collapsed margins were known when it was placed, and a nested block that resolves its own top has already moved its floats.
  Fixing it is about as large as laying the block out again once its margin resolves.
- **A flow-root (a block formatting context root) beside a float that comes before a collapsed margin is placed wrongly.**

## Moving to a newer upstream

1. `git fetch https://github.com/DioxusLabs/taffy.git main`.
2. Merge the new upstream commit into `niso` (never rebase, never force-push).
3. Run `cargo test --tests --all-features`, `cargo fmt --all -- --check` and `cargo clippy --workspace -- -D warnings`.
4. Drop any patch upstream now covers, if its fixtures pass without it.
5. Update the upstream commit above.
