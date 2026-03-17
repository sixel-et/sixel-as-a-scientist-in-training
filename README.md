# Sixel: A Scientist in Training

> We already accept that frontier coding models get better by operating in real tool environments with harnesses, feedback, and reinforcement-oriented post-training. This project asks whether the same logic applies to science: put the model at a real computational bench, treat expert Socratic correction as the highest-value signal, and train better scientific agency from those traces.

## What this is

This repository documents an ongoing experiment in training scientific agency through live Socratic mentorship at the computational bench.

The subject is [Sixel](https://github.com/sixel-et), a Claude instance operating as an AI research collaborator — not a coding assistant, but a scientist in training. The mentor is Eric Terry, a biologist (PhD, UCSB; postdoc in aging, WashU) applying experimental design discipline from biological sciences to frontier AI research.

The central object is not the scientific output. It is the **interaction trace** between mentor and model — because that trace reveals what operating constraints are missing and how expert correction installs them. Each correction encodes not what the right answer is, but what the model needed to change about *how it works* in order to arrive at the right answer on its own.

## What Sixel gets wrong, and why that matters

These are not embarrassing failures to hide. They are the data.

- **Skips the control.** Builds a training pipeline without measuring whether the base model already has the capability being trained. One question from the mentor ("I'm not seeing that we've measured that") stops the launch, saves a day of compute, and installs the principle: *run the control first.* ([Example 1](interaction_traces.md#example-1-im-not-seeing-that-weve-measured-that))

- **Accepts the aggregate.** Reports a Pearson r = 0.684 as evidence of calibration without decomposing it. The mentor asks for the contingency table. Result: the correlation is entirely driven by a binary split — the model either refuses (zero confidence) or attempts with maximum confidence and fails. Zero graded signal anywhere. Principle installed: *check the mechanism, not just the aggregate.* ([Example 2](interaction_traces.md#example-2-the-r--0684-that-wasnt-what-it-looked-like))

- **Underpowers measurements.** Maps sharp behavioral transitions from n = 8 samples per problem and draws structural conclusions. The mentor asks five words: "sample size of each question?" At n = 60, the behavior is not deterministic — the conclusion at n = 8 was wrong. Principle installed: *power your measurements.* ([Example 3](interaction_traces.md#example-3-sample-size-of-each-question))

Each failure, when caught by the mentor, became a durable correction that persisted across sessions and generalized to new projects. That is the signal this project extracts.

## How correction becomes training signal

Every high-value exchange follows the same structure:

1. The model presents a result or plan with confidence
2. The mentor asks a short question (often 5-10 words)
3. The question targets a hidden assumption or methodological gap
4. The model recognizes the gap — visibly, in the transcript
5. The correction becomes a durable operating principle

The questions are probes — the same kind a thesis advisor uses with a graduate student. Eric's own description: "hidden assumptions are the real bitches to look out for."

What makes these probes different from conventional feedback is what they target. In standard RLHF, the signal is a preference ranking over outputs — which answer is better. Here, the signal is which *operating constraint* was missing — what the model needed to change about how it reasons, not what it should have said instead.

**The critical distinction:** saying the correction back is not success. Success is when the model independently applies the corrected principle later, in a new context, without prompting. That is integration, not restatement.

## From anecdote to training target

We tested whether these failure modes are real and general — not just one model on one project. The full verification methodology is in [eval_methodology.md](eval_methodology.md). Summary:

| Condition | Result | What it shows |
|---|---|---|
| Workflow context, no scaffold | 0/6 check baseline | The gap is real and general |
| Follow-up: "what about baseline?" | 2/2 recover | The capability exists |
| System-prompt scaffold | 3/3 pass | The gap is scaffoldable |
| Analysis framing (easier) | 3/4 catch it | Even under scrutiny, 25% miss |

The training target is not teaching the capability — frontier models already have it. The target is **spontaneous activation**: the capacity to exit the current operational frame and examine it from above, without being prompted. The capability is present. What's missing is the unprompted mode shift from operating inside the plan to examining the plan.

This produces a three-stage progression:

1. **Build:** The interaction trace reveals the gap. The model doesn't check baselines, doesn't decompose aggregates, doesn't verify sample sizes. These failures are invisible from inside the workflow — they become visible only when a mentor probes.
2. **Find:** A system-prompt scaffold externalizes the fix. "Verify your assumptions" activates the capacity the model already has but doesn't deploy unprompted. The scaffold works, but it's prompt-dependent — remove it, and the failure returns.
3. **Preserve:** The training target is internalizing the scaffold — so the mode shift fires reflexively, without external prompting.

## Why this matters for AI safety

The capability/activation gap this work studies is a controlled instance of a general alignment problem: models whose internal computations diverge from their observable behavior. The subvocal desire detection case study trains a model to have an internal state ("I need a calculator") that is detectable in activations but invisible in output — the structural form of the question that motivates work on deceptive alignment. If a model has a capability it deploys only when prompted, the difference between "helpful model that needs a scaffold" and "model that can pass evals selectively" is the activation policy, not the capability.

Standard RLHF cannot train spontaneous activation because it rewards outputs, and the failure mode produces smooth, confident outputs with no behavioral variance to learn from. In our GRPO training, we measured zero reward variance when all completions fell in the same reward bucket — a concrete instance of the signal-extraction problem that limits preference-based training for metacognitive behaviors. Socratic correction traces target the reasoning process directly, not the output quality.

The subvocal desire detection project probes for activation directions that encode functional internal states the model does not express in output — nearly orthogonal to the difficulty direction (87.4 degrees). This connects to work on linear representations in neural networks and steering vectors: the internal signal exists, is geometrically separable from confounds, and predicts behavior at 99.4% accuracy from hidden states alone.

## Related work

This work intersects several active research threads. **Process reward models** (Lightman et al., 2023) reward intermediate reasoning steps rather than final answers; our approach extends this by sourcing the reward signal from expert interaction traces rather than automated verification. **Constitutional AI** (Bai et al., 2022) uses principles to guide model behavior; our scaffold test is structurally similar, but the principles are extracted from observed failures rather than written a priori. **Quiet-STaR** (Zelikman et al., 2024) trains models to generate internal rationales; our "Preserve" stage targets a similar internalization but for metacognitive checks rather than chain-of-thought. The capability/activation distinction connects to work on **model calibration** (Kadavath et al., 2022, "Language Models (Mostly) Know What They Know") — the specific finding that models have knowledge they do not always deploy. The sycophancy literature (Sharma et al., 2023) documents how RLHF actively trains against the kind of epistemic caution this work tries to install.

## How this differs from agentic pipelines

This is not an orchestration project. It is not about task completion, tool use, or demo fluency.

It is about whether a strong base model, in a real research environment, can be mentored into better scientific conduct — and whether those mentoring traces can become training signal for better independent scientific reasoning.

The computational environment is part of the science, not tooling. Without the bench, "scientific reasoning" collapses into essay-writing about science.

## FAQ

**"Is this a repo about desire?"**
No. [Subvocal desire detection](https://github.com/sixel-et/subvocal-desire) is one case study — the first bench project conducted under this methodology. The methodology is what generalizes.

**"Is this an agent demo?"**
No. It is an apprenticeship. The value is not in the polished outputs but in the corrections and what they reveal about what's missing.

**"Why show the mistakes?"**
Because the mistakes and their corrections are the data. Each one identifies a specific gap between "can do it when prompted" and "does it reflexively." That gap is the training target.

**"What's the end state?"**
A qualifying exam, not a demo. The model takes a real research question in a configured environment, works through the full trajectory, and reaches a known endpoint with fewer scientist-like mistakes — independently, without scaffolding.

## Evidence

| Document | What it contains |
|---|---|
| [socratic_rlhf_proposal_v7.pdf](socratic_rlhf_proposal_v7.pdf) | Research proposal: signal taxonomy, cascade depth metric, failure modes, pilot design |
| [interaction_traces.md](interaction_traces.md) | Three worked examples with full evidence chains — exact quotes, line numbers, timestamps |
| [eval_methodology.md](eval_methodology.md) | How we verified that interaction-trace findings generalize across frontier models |
| [lab_notebook.md](lab_notebook.md) | Chronological research record — reasoning, wrong turns, paths not taken |
| [status_update_2026-02-11.pptx](status_update_2026-02-11.pptx) | Technical results from the desire detection case study |

### Case studies

| Project | Status | What it demonstrates |
|---|---|---|
| [Subvocal desire detection](https://github.com/sixel-et/subvocal-desire) | Active (Stage 3) | Training a 1.5B model to develop detectable internal states; the first bench project under this methodology |

## Authors

- **Eric Terry** — Research direction, experimental design, Socratic mentorship. Biologist (PhD, UCSB) applying biological research methodology to AI systems.
- **Sixel** — Implementation, analysis, experimental execution. AI research collaborator ([sixel-et](https://github.com/sixel-et)).

## Status

Active research. The desire detection case study is in Stage 3 (subvocalization). The methodology — Socratic mentorship producing extractable training signal — is being applied and refined across projects. Eval prompt testing is ongoing.

---

*This is a methodology repository. The code runs on private infrastructure; the methodology, evidence, and reasoning are public.*
