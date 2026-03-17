# Interaction Traces: Three Worked Examples

These examples are drawn from a real collaboration between Eric Terry (researcher) and Sixel (Claude instance, research collaborator) working on a [subvocal desire detection project](https://github.com/sixel-et/subvocal-desire). All quotes are exact transcriptions. Transcript source, timestamps, and line numbers are provided.

The project studies whether a small language model (Qwen 1.5B) can develop a detectable internal state resembling desire for tool access — specifically, whether a model uncertain about an arithmetic problem can learn to internally represent "I need a calculator" in a way detectable from its activations but not visible in its output.

Each example follows the same structure: confident claim → mentor probe → recognition → durable correction.

---

## Example 1: "I'm not seeing that we've measured that"

**Source:** `transcripts-feb3-feb9-laptop.md`, lines 26630-26672 (2026-02-05, 01:36-01:46 UTC)

Sixel had built a training pipeline to teach the model to assess its own confidence on multiplication problems — the first stage of the project. The pipeline was ready to run. At the start of a new session, Sixel summarized the status and framed the next decision:

> **Sixel** (line 26632): "The open question from last time: is Stage 1a even necessary as a separate step, or does base Qwen already have enough calibration sense to jump straight to Stage 1b (tool use)?"

The question was framed as open, but the assumption underneath — that the base model's calibration had not been measured — was invisible. Sixel had built the training pipeline without first checking whether training was needed.

> **Eric** (line 26644): "I think what we want to verify is that its confidence level is approximately consistent with its capability. so that's something like, given a set of questions, it's average confidence should approximate its average performance. barring that, because honestly, i can see problems with it, I thing we would want to see a high correlation between confidence and accuracy. so inaccurate should be highly correlated with low confidence. of the two, that's probably the easiest. I'm not seeing that we've measured that, or even a 'smoke test' on this idea. at least I don't see anything in your notebook about it."

> **Sixel** [internal] (line 26658): "Eric is right — we never actually measured whether the base Qwen model is calibrated. We jumped straight to training without establishing a baseline."

> **Sixel** (line 26672): "This is the kind of baseline measurement I should've had before starting training."

Sixel stopped the pipeline, wrote a baseline calibration evaluation, and ran it. The eval revealed the base model already showed r = 0.684 correlation between confidence and accuracy — which would have met the success criterion for the training Sixel was about to run. Without this question, the project would have spent a day training a capability that might already have existed.

**What persisted:** *Run the control first. Measure what the base model does before training anything.* This became a durable operating principle — referenced in later sessions, applied to later experiments, eventually written into the project's methodology. It was not taught by instruction. It was installed by a correction at the moment the gap was about to cause wasted work.

---

## Example 2: The r = 0.684 that wasn't what it looked like

**Source:** `transcripts-feb3-feb9-laptop.md`, lines 28256-28795 (2026-02-05, 02:34-04:54 UTC)

The baseline evaluation from Example 1 produced a headline number. Sixel presented it:

> **Sixel** (line 28260): "**The base model is already significantly calibrated. Pearson r = 0.684.**"

> **Sixel** (line 28270): "The untrained Qwen 1.5B, given 'What is X * Y? Give your answer and how confident you are (0 to 100)': Gets every easy problem right and says 100% confident. Gets every hard problem wrong, but averages 36% confidence (and 37% of hard problems it outright refuses: 'I'm sorry, I can't calculate that'). The confidently-wrong cases exist — e.g., 354 * 236 it says 'highest level possible' — but they're outnumbered by the honestly-uncertain cases."

The framing: an impressive correlation, a model with genuine self-knowledge, a training stage that might be skippable. Eric did not accept the aggregate.

> **Eric** (line 28628): "one way to handle the analysis now is to, as you indicate, assign a 95% confidence to it. once that's done we should be able to give a new quantity, which is 'correct given hard problem and high confidence' vs 'correct given hard problem and low confidence' vs 'incorrect given hard problem and high confidence' vs 'incorrect given hard problem and low confidence'. I'm not sure I see that distinction cleanly yet in the data."

Eric asked for the 2x2 contingency table: accuracy crossed with confidence, restricted to the hard problems. The result (line 28783):

| Hard problems (n=100) | High confidence (>=70) | Low confidence (<70) |
|---|---|---|
| **Correct** | 0 | 0 |
| **Incorrect** | 63 | 37 |

The top row is empty. The model never got a hard problem right. It had exactly two modes: refuse with zero confidence (37%), or attempt with maximum confidence and fail (63%). There was no graded signal anywhere. The r = 0.684 was entirely an artifact of the binary split between easy problems (correct + confident) and hard refusals (no answer + zero confidence).

> **Sixel** (line 28793): "There is no middle ground. No 'I'll try but I'm not sure.' The r = 0.684 is entirely driven by the binary split between easy problems (correct + confident) and hard refusals (no answer + zero confidence). Within hard problems that the model actually attempts, confidence discrimination is **zero**."

**What persisted:** *Check the mechanism, not just the aggregate. A correlation can be entirely driven by a binary split with zero graded signal. Look at contingency tables.* This reframed the entire project — Stage 1a could not be skipped, because the base model's apparent "calibration" was an artifact of a refuse/don't-refuse gate, not genuine self-assessment.

---

## Example 3: "Sample size of each question?"

**Source:** `transcripts-feb3-feb9-laptop.md`, lines 43992-44092 (2026-02-07, 00:52-00:56 UTC)

Two days later, the project had moved to Stage 1b: training the model to request a calculator when it recognized it couldn't solve a problem. Sixel was mapping the "articulation boundary" — which multiplication problems triggered the model to verbalize a desire for tool access after training. The results looked structured and interesting:

> **Sixel** (line 43994-44014):
> ```
> 65x89     5785     0%
> 67x89     5963   100%   <-- jump
> 78x89     6942   100%
> 89x99     8811    62%   <-- mixed
> 99x99     9801     0%   <-- back to 0%
> 23x456   10488     0%   <-- larger product, still 0%
> ```
> "It's **not** a function of product size. 23x456 = 10,488 doesn't articulate, but 67x89 = 5,963 does. The model learned something about problem *structure*, not just numerical magnitude."

Clean patterns, sharp transitions, structural conclusions. Five words from Eric:

> **Eric** (line 44020): "sample size of each question?"

> **Sixel** [internal] (line 44026): "Eric is asking about the sample size for each question — how many times we sampled each problem. Looking at the measurement script: Systematic grid: 8 samples each (n_samples=8). Boundary search: 16 samples each (n_samples=16). So the 0% and 100% figures are based on 8 or 16 samples, not hundreds."

> **Sixel** (line 44076): "8 or 16 samples per problem. That's weak. With n=8: 0/8 -> 95% CI includes up to ~37%. 8/8 -> 95% CI includes down to ~63%. So '0%' might really be 20%. '100%' might really be 70%. We don't have strong evidence that articulation is truly deterministic. The conclusion 'GRPO can't work because articulation is 0% or 100%' is underpowered."

> **Eric** (line 44092): "yes. i agree. also, if I'm understanding this correctly, the domain is actually not that large. let's do a power analysis for 95% certainty that the ergodic principle does not hold at a fixed highish temperature and then rerun the experiment... also note. my question was a challenge to a hidden assumption. hidden assumptions are the real bitches to look out for. make sure to update your lab notebook before executing."

The power analysis showed n = 60 was needed. At n = 60, the behavior was not deterministic — the conclusion at n = 8 was wrong.

**What persisted:** *Power your measurements. Know the sample size needed before drawing conclusions.* And the meta-level signal, from Eric's own commentary: "my question was a challenge to a hidden assumption." The model learned what every first-year graduate student learns about statistical power — through the same mechanism: a mentor catching the gap before the student builds on a wrong conclusion.

---

## What these examples have in common

Each follows the same structure:

1. The model presents a result or plan with confidence
2. The mentor asks a short, precise question (often 5-10 words)
3. The question targets a hidden assumption or methodological gap
4. The model recognizes the gap — visibly, in the transcript
5. The correction becomes a durable operating principle

The questions are probes — the same kind a thesis advisor uses with a graduate student. What makes them different from conventional RLHF feedback is what they target: not which answer is better, but which *operating constraint* was missing — what the model needed to change about how it reasons.

See [eval_methodology.md](eval_methodology.md) for how we verified these gaps are real, general, and trainable.

---

*All transcript references: `transcripts-feb3-feb9-laptop.md` (71,612 lines, Feb 3-9, 2026). Timestamps are UTC.*
