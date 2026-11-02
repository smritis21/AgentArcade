# BALROG Agentic LLM Experiments

An interactive, data-backed dashboard and reproducible experiment artifacts for
three LLM agent designs evaluated on the BabyAI benchmark:

1. **Naive baseline** — choose one action from the current observation and
   recent history.
2. **Planning memory** — maintain a short plan and current subgoal while
   selecting actions.
3. **Bounded self-reflection** — let a critic review recent behavior when a
   loop is detected or a review is scheduled, then pass accepted advice to the
   actor.

The experiments compare a local **Qwen2.5 1.5B** model with **Gemini 3.5
Flash-Lite** across five BabyAI task types and seeds 42–46. The repository
contains the runners, submitted agent implementations, tests, reports,
comparison tables, and saved experiment outputs. The dashboard reads those
saved outputs locally; it does not make model or network requests.

> **Scope:** These are small, exploratory course experiments—not a reproduction
> of the complete BALROG paper or evidence of general reasoning ability.

## Dashboard

Open [`index.html`](index.html) in a modern browser. It is a standalone page
with no build step or server dependency. Use the filters and episode controls to
explore the recorded results, replay agent actions, and inspect observations,
prompts, responses, memory, and reflection events.

![BALROG experiment dashboard showing aggregate results and an episode replay](assets/dashboard.png)

The screenshot above is captured from the supplied website using the local,
saved experiment outputs.

## Results at a glance

Success means completing the mission within the action limit. Each model and
agent combination has 25 episodes: five task types and five seeds. The
dashboard and the stage-level `comparison.csv` files contain the detailed
results.

| Model | Naive baseline | Planning memory | Self-reflection |
|---|---:|---:|---:|
| Qwen2.5 1.5B (local) | 28% (7/25) | 8% (2/25) | 12% (3/25) |
| Gemini 3.5 Flash-Lite (cloud) | 84% (21/25) | 60% (15/25) | 88% (22/25) |

These are observed outcomes from a small sample, not statistically significant
claims. The planning treatment changes the prompt, memory, and parser together.
Reflection adds critic calls and computation, so it is not an equal-compute
comparison. Earlier results informed the later agent designs; see each
exercise's report for the limitations and comparison details.

## Repository layout

```text
.
├── index.html                 # Interactive, offline experiment dashboard
├── assets/
│   └── dashboard.png          # Screenshot embedded above
├── Ass2-1/                    # Naive baseline evaluation and saved runs
├── Ass2-2/                    # Planning-memory agent, tests, and results
└── Ass2-3/                    # Reflection agent, tests, and results
```

Each `Ass2-*` directory includes its own README, Python runner, evaluation
artifacts, and—where applicable—an experiment report. The saved `results/`
directories include episode-level data and model-call traces used by the
dashboard. Raw traces may contain prompts and model responses; review them
before redistributing or publishing any additional runs.

## Evaluation setup

The experiments were run with **Python 3.10.21** and the BALROG source checkout
at revision `b7afe79e3e4265811c1fa985ed7c95c4d1a11e3f5`. BALROG is an upstream
dependency and is not vendored in this repository. Follow its installation
instructions for the dependencies needed by the environments:

```bash
git clone https://github.com/balrog-ai/BALROG.git
git -C BALROG checkout b7afe79e3e4265811c1fa985ed7c95c4d1a11e3f5
python -m pip install -e ./BALROG
```

For a matching Python environment, create and activate a Python 3.10 Conda
environment before installing BALROG:

```bash
conda create -n balrog python=3.10 -y
conda activate balrog
```

The run folders record their package versions and configuration. See the
upstream BALROG installation guide and each exercise README for any additional
platform-specific setup.

### Model access

- **Local Qwen:** Install and start [Ollama](https://ollama.com/), then download
  the model with `ollama pull qwen2.5:1.5b`. The runner expects the local Ollama
  service at `localhost:11434`.
- **Gemini:** Set `GEMINI_API_KEY` in the shell that will run the experiment.
  In PowerShell, for example, use `$env:GEMINI_API_KEY = Read-Host "Gemini API key"`.
  Never commit API keys, `.env` files, or other credentials.

Saved results and the dashboard can be inspected without either model or API
credentials.

## Running experiments

Run commands from the repository root. Full evaluation runs use the
`baseline` profile; without that profile, the runners use a short diagnostic
configuration.

```bash
# Exercise 1: naive baseline
python Ass2-1/run_local.py --profile baseline --seeds 42 43 44 45 46

# Exercise 2: planning/subgoal memory
python Ass2-2/run_exercise2.py --agent custom --profile baseline

# Exercise 3: bounded self-reflection
python Ass2-3/run_exercise3.py --agent reflection --profile baseline

# View saved run status without making API calls
python Ass2-3/status.py
```

To run a cloud evaluation, add `--provider gemini` to the corresponding
command after configuring `GEMINI_API_KEY`. For example:

```bash
python Ass2-2/run_exercise2.py --provider gemini --agent custom --profile baseline
```

Exercise 2's runner loads its submitted agent from the BALROG checkout. Install
that copy before running:

```bash
cp Ass2-2/agent/custom.py BALROG/balrog/agents/custom.py
```

Exercise 3 registers its reflection agent in its runner and does not require
copying an agent file into BALROG. To resume an interrupted evaluation, pass the
existing run directory to the stage runner:

```bash
python Ass2-3/run_exercise3.py --resume Ass2-3/results/YOUR_RUN_FOLDER
```

Replace `YOUR_RUN_FOLDER` with the actual directory name. Completed episodes
are skipped; an interrupted episode restarts. Consult the stage README files
for supported providers, agent options, and detailed run behavior.

## Tests

The tests are organized by exercise. With the experiment environment and
BALROG dependencies installed, run:

```bash
python -m pytest Ass2-1/test_cloud_runner.py -q
python -m pytest Ass2-2/test_agent.py Ass2-2/test_runner.py Ass2-2/test_comparison.py Ass2-2/test_package.py -q
python -m pytest Ass2-3/test_reflection.py Ass2-3/test_runner.py Ass2-3/test_artifacts.py -q
```

The individual stage READMEs also document how to package the submitted
agents.

## Experiment details

- **Benchmark tasks:** `goto`, `pickup`, `open`, `putnext`, and
  `pick_up_seq_go_to`.
- **Seeds:** 42–46, with one episode per task and seed (25 episodes per
  model/agent combination).
- **Action limits:** 64 for `goto` and `pickup`; 128 for the other tasks.
- **Observation:** Text-only; no images; 16 recent observations.
- **Baseline:** One model call selects an action.
- **Planning memory:** Adds bounded plan and subgoal fields to the agent's
  persistent episode memory.
- **Self-reflection:** A bounded critic reviews recent transitions after
  configured loop or scheduled-review triggers; advice is passed to the actor.

Environment seeds do not make model responses deterministic. Provider-side
model versions may change over time, and the historical baseline runs may not
use the exact same provider model version as later experiments. Review
`Ass2-2/REPORT_RESULTS.md` and `Ass2-3/REPORT_RESULTS.md` for further caveats,
task-level comparisons, and invalid-action and critic-call counts.
