# Changes to RPent

`rpent_f70d720.patch` applies to RPent at commit `f70d720` and gives the RPent code all evaluations of the paper used
(8 files, +303 / -9 lines). The planner's tools and RPent's control loop are not changed.

```bash
git clone https://github.com/RLinf/RPent.git && cd RPent
git checkout f70d720
git apply --check /path/to/AgenticRoboRL/patches/rpent/rpent_f70d720.patch
git apply /path/to/AgenticRoboRL/patches/rpent/rpent_f70d720.patch
git add -A && git write-tree        # 11c44fcdedab126962be13ed3f477e0212932323
```

| Change | Files |
|---|---|
| `--knowledge-profile proven`: the zero-shot setting of Harness VLA. The planner gets the cross-task "proven levers" text whole and the guides, but no task memory and no global memory; its file tools can read only the guides and the episode's own output directory | `robots/libero/prompts/knowledge_eval.py` (new), `robots/libero/prompt_bundle.py`, `robots/libero/robot_spec.py`, `robots/libero/toolkit.py`, `rpent/cli/main.py`, `rpent/cli/dashboard.py` |
| `--no-send-back-thinking`: the planner's earlier reasoning is not sent back to the model in later turns; `--temperature`, `--top-p`, `--top-k`, `--presence-penalty`, `--generation-seed` passed to the model server | `rpent/planner/api_loop.py`, `rpent/planner/base.py`, `rpent/cli/main.py`, `rpent/cli/dashboard.py` |
| The transcript records `solved` (the simulator's success flag) and `agent_error`; `rl/adapters/rpent_libero.py` reads the reward from them | `rpent/cli/main.py` |

The RL rollouts of the paper ran on a later version of these files that additionally recorded simulator state for
an experiment not reported in the paper (off unless an environment variable is set; nothing of it reaches the planner).
That recording is not part of this patch.

sha256 of `rpent_f70d720.patch`: `74380a09251aa42f516457b1f410415d7d7cd1aee84d661886c164e18b0860b7`

## Optional: concurrency fix (upstream PR #150 backport)

RPent at `f70d720` rejects overlapping tool calls: when the planner issues several tool calls in one
turn, the toolkit's single-operation lock fails all but one of them with `another tool operation is
still active`, and the planner spends extra turns recovering. Upstream fixed this in
[RLinf/RPent#150](https://github.com/RLinf/RPent/pull/150) (merged) by serializing a turn's tool
calls at the caller side. `rpent_f70d720_concurrency_fix.patch` backports that fix to `f70d720`
(2 files, +9 / -4 lines; the source changes are line-identical to the upstream fix, only the
insertion point of one keyword argument differs so the patch applies cleanly here).

Apply it (optionally) on bare `f70d720`, or on top of `rpent_f70d720.patch`:

```bash
git apply /path/to/AgenticRoboRL/patches/rpent/rpent_f70d720_concurrency_fix.patch
```

**Note on reproducibility:** every number in the paper (evaluations and RL training) was produced
WITHOUT this fix, i.e. with the concurrency bug present — the planner's recovery turns after a
rejected call are part of the recorded trajectories. To reproduce the paper exactly, do not apply
this patch. To run new experiments without the spurious failures, apply it.

sha256 of `rpent_f70d720_concurrency_fix.patch`: `5b81f9e8acb5ce18ec76513eb97dbb35269499be1cba70aa1c909cd8b614a97e`
