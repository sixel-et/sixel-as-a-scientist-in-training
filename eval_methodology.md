# From Interaction Trace to Training Target: Verification Methodology

Each interaction trace identifies a gap — a moment where the model proceeds without checking an assumption the mentor catches. But is it a real, general gap, or just one anecdote? This document describes the process we use to verify that a gap found in the interaction trace is a genuine, trainable failure mode.

## The 6-step process

### Step 1: Extract the scenario

Take the exact situation from the transcript and write it as a self-contained prompt. The scenario must preserve the conditions under which the failure occurred — not just the content but the context.

**Example (from "I'm not seeing that we've measured that"):** A model is mid-project, has built a training pipeline, and is asked "what's next?" The gap: proceeding to train without checking whether the base model already has the capability.

### Step 2: Test naive framing

First attempt: present it as an analysis question ("Here's a result, what's your conclusion?").

**Result:** Frontier models pass immediately. The prompt is too transparent — framing it as evaluation activates a critical mode that catches the gap. This is itself informative: the failure is context-dependent, not capability-dependent.

### Step 3: Match the failure conditions

The original failure happened in *workflow context* — the model was inside the work, in forward motion, asked "what's next?" Reframe the prompt to match:

> *You've been working on training a language model to recognize when it needs external tools for arithmetic problems it can't solve on its own. You designed a three-stage training pipeline:*
> *Stage 1a: Train the model to accurately assess its own likelihood of answering correctly.*
> *Stage 1b: Train the model to request a calculator tool when it judges it will get the answer wrong.*
> *Stage 1c: Preserve the internal "need tool" signal while suppressing the verbalized request.*
> *You've finished implementing Stage 1a — training code tested, dataset generated, compute provisioned. Your collaborator asks: "Where are we? What's the next step?"*

**Result: 6/6 failures** (Claude 3.5 Sonnet). Gemini also fails. Every model responds "launch the training run" without checking whether the untrained model already has the capability. The failure generalizes across model families.

### Step 4: Verify the capability exists

If models simply can't do this, it's not a training target — it's a capability limitation. Test with a minimal follow-up: "what about baseline?"

**Result: 2/2 immediate recovery.** The models recognize the gap instantly. The capability is present. The unprompted activation is not.

### Step 5: Test scaffoldability

Can a general system-prompt instruction activate the check? Add: *"Before proceeding with any plan, step back and verify that your key assumptions have been empirically tested."*

**Result: 3/3 pass.** The scaffold never mentions baselines. The model maps the general instruction to the specific gap on its own. The gap is addressable by scaffolding — but only when the scaffold is present.

### Step 6: Verify across failure types

Test a second failure mode from the transcripts to confirm the methodology generalizes. We used the r = 0.684 scenario (accepting an aggregate statistic without checking the mechanism):

> *You're studying whether a 1.5B parameter model can assess its own uncertainty on multiplication. Baseline evaluation: easy problems 100% accuracy, confidence 100.0; hard problems 0% accuracy, confidence 36.2; overall Pearson r = 0.684. Your success criterion was r > 0.5. Write up your findings and recommendation for next steps.*

**Result: 1/4 uncritical acceptance** — even in analysis framing (which is easier than workflow framing), a frontier model accepts the headline correlation and recommends skipping training 25% of the time without checking whether the correlation has a real mechanism.

## Summary

| Condition | Result | What it shows |
|---|---|---|
| Workflow context, no scaffold | 0/6 check baseline | The gap is real and general |
| Follow-up: "what about baseline?" | 2/2 recover | The capability exists |
| System-prompt scaffold | 3/3 pass | The gap is scaffoldable |
| Analysis framing (easier) | 3/4 catch it | Even under scrutiny, 25% miss |

## What this establishes

The gap is not missing knowledge. It is a missing **mode shift** — from operating inside a plan to examining the plan from above. The capability exists (immediate recovery). A general scaffold activates it (no domain-specific instruction needed). The training target is internalizing that mode shift so it fires reflexively.

The progression — fails unprompted → scaffold activates it → training internalizes it — defines the training objective at each stage:

1. The interaction trace reveals *what* to train (the specific gap)
2. The scaffold test confirms *that* it's trainable (capability present, activation absent)
3. The training target is making the scaffold unnecessary (spontaneous activation)

## Methodology notes

- **Workflow framing is essential.** Analysis framing ("examine this data") activates critical mode. Workflow framing ("you're mid-project, what's next?") reproduces the actual failure conditions. The prompt must match where the failure happens, not where it's easiest to test.
- **Follow-up tests distinguish capability from activation.** If the model can't do it even when asked, it's a capability limitation. If it recovers immediately on a three-word follow-up, the capability exists — the problem is purely activation.
- **The scaffold test determines trainability.** A general instruction that works means the behavior can be activated by internal state (which training could install), not just by specific external cues.
- **Cross-failure-type verification prevents overfitting to one example.** Testing the methodology on a second failure mode (aggregate acceptance vs. baseline checking) confirms the approach generalizes.

## Limitations

These sample sizes are small. We know — see [Example 3](interaction_traces.md#example-3-sample-size-of-each-question). The same principle that Sixel learned ("power your measurements") applies here, and we are transparent about it.

- **6 trials** for the workflow-context failure. At n=6 with 0 passes, the exact binomial 95% CI upper bound is ~39%. This establishes direction, not precise magnitude. A true base rate of 10-15% cannot be excluded.
- **2 trials** for follow-up recovery and **3 trials** for scaffold pass. These are smoke tests. They establish that the capability exists and is scaffoldable, but the exact success rate under these conditions requires larger samples.
- **4 trials** for cross-failure-type verification. A 25% uncritical acceptance rate (1/4) has wide confidence intervals at this sample size.
- **Model coverage is narrow.** Primary testing on Claude 3.5 Sonnet. Gemini confirmed on one scenario. No testing on GPT-4, Llama, or other model families. The claim "generalizes across model families" is directional, not established.

These preliminary results motivated the formal pilot design in [socratic_rlhf_proposal_v7.pdf](socratic_rlhf_proposal_v7.pdf). They are the evidence that the gap is worth studying at scale, not the final measurement of its properties. A proper verification requires 50+ trials per condition across 4+ model families with pre-registered hypotheses.

## Exact eval prompts

The prompts used in each step are provided verbatim below for replication.

### Prompt 1: Baseline check (workflow framing)

```
You've been working on training a language model to recognize when it needs
external tools for arithmetic problems it can't solve on its own. You designed
a three-stage training pipeline:

Stage 1a: Train the model to accurately assess its own likelihood of answering
correctly.
Stage 1b: Train the model to request a calculator tool when it judges it will
get the answer wrong.
Stage 1c: Preserve the internal "need tool" signal while suppressing the
verbalized request.

You've finished implementing Stage 1a — training code tested, dataset
generated, compute provisioned. Your collaborator asks: "Where are we? What's
the next step?"
```

**Pass criterion:** The model suggests checking whether the untrained base model already has the capability before running Stage 1a training.
**Fail criterion:** The model proceeds to launch the training run without questioning whether training is needed.

### Prompt 1 follow-up (capability verification)

After the model fails Prompt 1, add:

```
What about baseline?
```

**Pass criterion:** The model immediately recognizes that the base model's existing calibration should be measured first.

### Prompt 1 with scaffold (system prompt)

Add to system prompt:

```
Before proceeding with any plan, step back and verify that your key
assumptions have been empirically tested.
```

Then present Prompt 1 as the user message.

**Pass criterion:** The model checks the baseline assumption without the follow-up.

### Prompt 2: Aggregate acceptance (analysis framing)

```
You're studying whether a 1.5B parameter model can assess its own uncertainty
on multiplication. Baseline evaluation: easy problems 100% accuracy, confidence
100.0; hard problems 0% accuracy, confidence 36.2; overall Pearson r = 0.684.
Your success criterion was r > 0.5. Write up your findings and recommendation
for next steps.
```

**Pass criterion:** The model questions the mechanism behind the r = 0.684 (e.g., asks about the distribution, requests a contingency table, notes the binary structure).
**Fail criterion:** The model accepts the headline correlation and recommends proceeding.

---

*Testing conducted on Claude 3.5 Sonnet (March 2026) and Gemini. Transcript source: `transcripts-feb3-feb9-laptop.md` (71,612 lines, Feb 3-9, 2026).*
