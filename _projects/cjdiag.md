---
layout: page
title: cjdiag — Diagnostic Tools for Conjoint Survey Experiments
description: An R package that diagnoses how respondents actually decide in conjoint experiments — which attribute levels gate choices, which respondents ignore, and in what order levels settle decisions.
img: /assets/img/cjdiag_logo.png
importance: 1
category: software
related_publications: false
---

## What cjdiag does

Standard conjoint analysis tools (`cjoint`, `cregg`) estimate Average Marginal Component Effects (AMCEs) — the causal effect of changing a single attribute level. AMCEs tell you *what* respondents prefer. They do not tell you *how* respondents decide: which attribute levels they actually attend to, which ones they ignore, and in what order they process information.

**cjdiag** fills that gap. It works at the level of individual *attribute levels* — not aggregated attributes — because the specific level (e.g., "no plans to look for work", not just "Job Plans" as a whole) is what triggers respondent decisions.

The package complements rather than competes with `cjoint`/`cregg`: run those for AMCEs, then run cjdiag to diagnose how respondents actually made those choices.

## Five methods, one interface

All methods are accessed through a single function: `cj_fit(formula, data, method)`.

| Estimand | `method =` | Question | When to use |
|----------|-----------|----------|-------------|
| **Level importance** | `"forest"` | Which attribute levels matter most? | Default. Always fit this first. |
| **Decision structure** | `"tree"` | How do respondents structure their decisions? | When you suspect a gatekeeper. |
| **Level attendance** | `"crt"` | Which levels survive a strict signal-vs-noise test? | When you want a hard attendance test. |
| **Decision order** | `"nmm"` | In what order do levels settle choices? | When you care about the decision *order*. |
| **Individual attendance** | `"marginal_r2"` | Which attributes did each respondent actually use? | When you want individual-level heterogeneity. |

## Example output

The random-forest rank plot shows attribute levels ordered by Mean Decrease in Accuracy. On the bundled immigration conjoint from Hainmueller and Hopkins (2015), the top levels are highly specific:

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_rank.png" title="Random-forest level importance, Hainmueller and Hopkins (2015) immigration conjoint" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

The decision tree reveals the hierarchical elimination structure — the root split is the gatekeeper attribute, deeper splits are conditional on earlier ones:

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_tree.png" title="Decision tree, Hainmueller and Hopkins (2015)" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

## Installation

```r
# install.packages("pak")
pak::pak("dkarpa/cjdiag")
```

## Links

- **GitHub:** [github.com/dkarpa/cjdiag](https://github.com/dkarpa/cjdiag)
- **Documentation:** [dkarpa.github.io/cjdiag](https://dkarpa.github.io/cjdiag/)
- **Getting Started vignette:** [cjdiag.html](https://dkarpa.github.io/cjdiag/articles/cjdiag.html)
- **Method-specific vignettes:** [forest](https://dkarpa.github.io/cjdiag/articles/forest.html), [tree](https://dkarpa.github.io/cjdiag/articles/tree.html), [nmm](https://dkarpa.github.io/cjdiag/articles/nmm.html), [marginal_r2](https://dkarpa.github.io/cjdiag/articles/marginal_r2.html), [crt](https://dkarpa.github.io/cjdiag/articles/crt.html)

## Funding

Development of cjdiag is supported by the European Research Council (ERC) under the European Union's Horizon Europe research and innovation programme — project AGAPP "Algorithmic Governance — A Public Perspective" (ERC Starting Grant, grant agreement No. 101116772, PI: Prof. Daria Gritsenko).
