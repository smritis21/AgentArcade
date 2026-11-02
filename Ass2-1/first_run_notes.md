# Initial checks — 30 September 2026

These are our own local installation checks, separate from the TA's supplied log.

- Python: 3.10.21 in conda environment `balrog`.
- BALROG commit: `b7afe79e3e4265811cfa985ed7c95c4d1a11e3f5`.
- Server: Ollama at `http://localhost:11434/v1`.
- Model: `qwen2.5:1.5b`; Ollama reported Q4_K_M quantization and digest
  `65ec06548149b04c096a120e4a6da9d4017ea809c91734ea5631e89f96ddc57b`.
- Agent: unchanged `naive`, text history 16, no images, temperature 1.0,
  output limit 128 tokens, one worker.
- Task: `BabyAI-MixedTrainLocal-v0/goto`, seed 42, maximum 20 actions.

| Check | Actual actions | Progression |
| --- | ---: | ---: |
| Direct upstream `eval.py` run | 7 | 100% |
| Starter script with prompt recording | 20 | 0% |

Both completed without an execution error. The second episode hit the 20-action
limit without completing the mission. Twenty saved model-call records matched its
20 action steps; no invalid action candidates were reported. Configuration, raw
responses, trajectories and the matrix were verified on disk.

The second run's mission was to go to the yellow key. Its trace contains pairs
of opposite turns (steps 0–1 and 3–4, zero-based), two `pick up` actions and one
`drop` action. At the final decision the key was still one step left and four
steps forward. This suggests inefficient navigation in this particular trace;
it does not establish how often this model behaves that way.

The two outcomes illustrate why one successful episode is only an integration
check. The environment seed was fixed, but model sampling was not seeded and
temperature was 1.0. Multiple tasks, seeds and repeated model runs are needed
before drawing performance or reasoning conclusions. The 20-step cap also limits
what can be concluded about failure under the normal environment action budget.

Direct evaluation artifacts are under `results/standard_smoke/`.
The recorded run is under `results/20260930_154531_616918_smoke_qwen2.5_1.5b/`.
The earlier `results/installation/` run used a dummy agent and is not an LLM result.
