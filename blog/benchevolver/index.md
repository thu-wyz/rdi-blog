---
layout: blog
title: "BenchEvolver: Frontier Task Synthesis via Solution-Centric Evolution"
---

<img src="framework.png" alt="BenchEvolver framework overview" class="cover-image" style="height: 100%; padding: 15px 15px 15px 30px;">

# BenchEvolver: Frontier Task Synthesis via Solution-Centric Evolution

<div class="author-info">
<strong>
    <a href="https://thu-wyz.github.io/">Yangzhen Wu*</a>,
    <a href="https://aaron-jx-li.github.io/">Aaron J. Li*</a>,
    Wenjie Ma, Li Cao, Ziheng Zhou, Mert Cemri, Shu Liu,
    Yuran Xiu, Chenxiao Yan, Haikun Zhao, Bin Yu,
    Ion Stoica&dagger;, Dawn Song&dagger;
</strong>
<br>
University of California, Berkeley &nbsp;·&nbsp; Institute for Interdisciplinary Information Sciences, Tsinghua University
<br>
(* equal contribution, &dagger; equal advising)
<br>
June 13, 2026
<br>
<em>(Est. 5 minutes read &mdash; more details on the
<a href="https://benchevolver.github.io/" target="_blank">project page</a>,
<a href="https://arxiv.org/abs/2606.01286" target="_blank">paper</a>,
<a href="https://github.com/thu-wyz/BenchEvolver" target="_blank">code</a>, and
<a href="https://huggingface.co/BenchEvolver" target="_blank">dataset</a>)</em>
</div>


*Static benchmarks are dying — they get saturated quickly by frontier models* For example, on LiveCodeBench, today's frontier models already score over **99%** on the newest easy problems and above **90%** on average. Once a benchmark saturates, it loses both evaluation and training value. It can no longer separate strong models, and it provides little signal for further improvement. But manually curating harder tasks is slow, expensive, and always one step behind the models. Evaluation should instead **co-evolve with the models it measures.**

## TL;DR

**BenchEvolver** automatically turns problems that models have already mastered into substantially harder, fully verified ones. The key shift is direction: instead of writing a new problem and hoping it is hard, BenchEvolver **evolves a reference solution first**, then builds the problem statement and tests.

Because each task is selected by whether models actually fail it—not by how difficult it appears—the resulting problems stay challenging even for the model that created them. No stronger teacher is required. This unlocks two things:

* **Upgraded benchmarks.** We curate **LiveCodeBench-Plus**, a 91-problem benchmark combining evolved tasks with difficult original LCB-v6 problems. Frontier-model Pass@1 ranges from **27.5% to 62.6%**, restoring clear separation among strong coding models.

* **Self-improvement.** Evolved tasks are not just harder benchmark items; they become training signal. Using **gpt-oss-20b** as both evolver and target, RL on evolved tasks improves held-out coding performance beyond training on the original seeds alone, with **70.7% more gain on LiveCodeBench v6 Hard** and **34.8% more gain on LiveCodeBench-Pro Easy**.

The loop closes: **models generate hard tasks → learn from them → get stronger.**


## One mutation, a whole new problem

The fastest way to see how BenchEvolver works is to watch it turn an easy problem into a hard one. Instead of writing a new problem and hoping it's difficult, it starts from a **known-correct solution**, mutates it into something structurally different, and only then writes the problem and tests to match:

> **working solution → evolved solution → new problem + tests**

Change the computation, and a familiar-looking problem can suddenly demand an entirely new algorithm. The same move works across two very different coding domains:

<img src="Example.png" alt="Two worked examples — Copy Arrays → XOR-Linked Sequence (LiveCodeBench), and RK4 Integrator → Fit ODE Trajectory (SciCode)" class="content-image" style="height: 540px; padding: 10px;">

**Competitive programming.** A counting problem solved by a simple linear scan becomes much harder with one tweak — swapping a "+" for an **XOR**. That small change shatters the clean structure the easy solution relied on, and the model now needs a fundamentally different algorithm. The pass rate drops from 8/8 to 4/8.

**Scientific computing.** A textbook simulation that runs a physical system *forward* is inverted into the much harder question of working *backward* — recovering the hidden parameters that would produce a handful of noisy observations. Same setting, a qualitatively tougher problem.

The surface story stays familiar; the computation underneath jumps to a different world. And the difficulty is *real* — it's baked into a solution that genuinely requires it, not bolted on as surface complexity.

## How it works: a loop that learns

Turning that one trick into a dependable pipeline takes more than a clever prompt. BenchEvolver runs an evolutionary loop with a memory, so each round builds on the last instead of just rerolling the dice:

<img src="framework.png" alt="BenchEvolver framework" class="content-image" style="height: 420px; padding: 10px;">

- **Proposer** mutates a solution into a harder one and writes a complete, runnable task around it.
- **Evaluator** throws out *fake* difficulty — checking that everything is consistent and that real models actually struggle.
- **Memory** remembers what worked, what failed, and which kinds of problems have already been tried, steering the search toward fresh, genuinely hard directions.

Three principles keep the output honest:

- **Generate in solution space.** Change the computation first, so difficulty is built into the task rather than painted onto its surface.
- **Verify by agreement.** Several independent checks — not a single AI judge — must agree the problem, solution, and tests all describe the same task.
- **Select by real failure.** Difficulty is *measured*, not assigned: a task survives only if models genuinely fail it more often than its original.

The result is a steady supply of verified, diverse, hard problems — with no human writing them and no stronger model to lean on.

## The problems are genuinely harder

Across both domains and every model we tried, evolved problems consistently cut pass rates relative to their originals. The most telling part: **each model also struggles on the problems it evolved itself** — the signature of *self-challenging* generation. The model is probing the exact frontier where its own ability runs out, not a strong teacher spoon-feeding a weaker student.

<img src="lcb_difficulty.png" alt="LiveCodeBench seed vs evolved Pass@1" class="content-image" style="height: 360px; padding: 10px;">

<img src="scicode_difficulty.png" alt="SciCode seed vs evolved Pass@1" class="content-image" style="height: 360px; padding: 10px;">

And the difficulty is more than skin-deep. Six competitive-programming experts (Codeforces master / IOI / ICPC level) reviewed the evolved problems by hand: they rated them more novel and much harder — yet just as *clear* as the originals. So this is real difficulty, not obfuscation. It also broadens *what* gets tested. The originals lean on a few familiar patterns; the evolved problems spread across a far wider range of advanced algorithms and data structures, growing the number of distinct algorithmic categories from **19 to 30**. BenchEvolver doesn't just turn up the difficulty dial — it expands the map.

<img src="algo_shift.png" alt="Algorithm category shift from seed to evolved" class="content-image" style="height: 320px; padding: 10px;">

## LiveCodeBench-Plus: a benchmark that discriminates again

We took the best evolved problems — each vetted by experts — and combined them with the hardest surviving originals into **LiveCodeBench-Plus**, a 91-problem benchmark for frontier coding models. On the hardest split, average pass rates fall from **87% to 46%**, and the strongest models now spread cleanly from **27.5% to 62.6%**. A benchmark that had gone quiet can tell models apart again.

| Model | Provider | Pass@1 |
|---|---|---|
| GPT-5.5 | OpenAI | 62.6 |
| Gemini-3.1-Pro | Google | 59.1 |
| GPT-5.4 | OpenAI | 54.1 |
| Gemini-3.5-Flash | Google | 50.0 |
| Qwen-3.7-Max | Alibaba | 47.5 |
| Gemini-3-Flash | Google | 40.1 |
| GPT-5.4-mini | OpenAI | 29.6 |
| DeepSeek-V4-Pro | DeepSeek | 27.5 |

## Self-improvement: harder problems make better teachers

Here's the part we find most exciting. If a model can generate problems that are genuinely hard *for itself*, those problems should be exactly what it needs to *learn* from. So we closed the loop: take a model, have it evolve problems it already solves into harder versions, then train it on those self-made challenges.

<img src="RL_curve1.png" alt="RL training curves: training on evolved problems beats training on the original seeds across three held-out benchmarks" class="content-image" style="height: 300px; padding: 10px;">

It works. Training on evolved problems improves held-out coding performance well beyond training on the original problems alone. On LiveCodeBench v6 Hard, adding evolved tasks gives up to +8.7 Pass@1 points, a 70.7% larger gain than seed-only training. On LiveCodeBench-Pro Easy, it gives 34.8% more gain than seed-only training.

The improvement also transfers to a separate evolved benchmark built by a different model, where evolved-task training gains +7.77 points and beats seed-only training by +4.56 points. So this is not just memorizing look-alike tasks. The model finds its own weaknesses, turns them into verified training signal, and learns from them.

## Why it matters, and what's next

Any fixed benchmark eventually saturates — and the better our models get, the faster it happens. BenchEvolver points to a different way of thinking about evaluation: not a frozen dataset that goes stale, but a **living benchmark** — a pipeline that keeps generating, verifying, and calibrating fresh challenges against whatever the frontier looks like today.

And because those same verified challenges can also *train* the models that fail them, evaluation and training stop being separate stages. They fuse into one loop: the problems that expose a model's limits become the problems that help it move past them.

For the full method, validation protocols, and experiments, see the [paper](https://arxiv.org/abs/2606.01286), [code](https://github.com/thu-wyz/BenchEvolver), and [dataset](https://huggingface.co/BenchEvolver).

#### Citation

```bibtex
@misc{wu2026benchevolverfrontiertasksynthesis,
      title={BenchEvolver: Frontier Task Synthesis via Solution-Centric Evolution},
      author={Yangzhen Wu and Aaron J. Li and Wenjie Ma and Li Cao and Ziheng Zhou and Mert Cemri and Shu Liu and Yuran Xiu and Chenxiao Yan and Haikun Zhao and Bin Yu and Ion Stoica and Dawn Song},
      year={2026},
      eprint={2606.01286},
      archivePrefix={arXiv},
      primaryClass={cs.SE},
      url={https://arxiv.org/abs/2606.01286},
}
```
