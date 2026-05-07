---
layout: page
title: cjdiag — Diagnostic Tools for Conjoint Survey Experiments
description: An R package that recovers the decision process AMCEs are agnostic about by design. Which attribute levels gate choices, which respondents ignore, and in what order levels settle decisions.
img: /assets/img/cjdiag_logo.png
importance: 1
category: software
related_publications: false
---

## What cjdiag does

Standard conjoint analysis tools (`cjoint`, `cregg`) estimate Average Marginal Component Effects (AMCEs), the causal effect of changing a single attribute level. AMCEs are valid and process-agnostic by design (Hainmueller, Hopkins, and Yamamoto 2014). They tell you what respondents prefer on average. They are deliberately silent on how respondents reach the choice, which attribute levels they actually attend to, which ones they ignore, and in what order they process information.

**cjdiag** is the tool I built to fill that gap. It works at the level of individual *attribute levels* rather than aggregated attributes, because the specific level (e.g., "no plans to look for work" rather than "Job Plans" as a whole) is what triggers respondent decisions.

The package complements `cjoint` and `cregg`. Run those for AMCEs, then run `cjdiag` to recover the decision process AMCEs leave on the table.

## Five methods, one interface

All methods are accessed through a single function, `cj_fit(formula, data, method)`.

| Estimand | `method =` | Question | When to use |
|----------|-----------|----------|-------------|
| **Level importance** | `"forest"` | Which attribute levels matter most? | Default. Always fit this first. |
| **Decision structure** | `"tree"` | How do respondents structure their decisions? | When you suspect a gatekeeper. |
| **Level attendance** | `"crt"` | Which levels survive a strict signal-vs-noise test? | When you want a hard attendance test. |
| **Decision order** | `"nmm"` | In what order do levels settle choices? | When you care about the decision order. |
| **Individual attendance** | `"marginal_r2"` | Which attributes did each respondent actually use? | When you want individual-level heterogeneity. |

## Example output

The random-forest rank plot shows attribute levels ordered by Mean Decrease in Accuracy (MDA). On the bundled immigration conjoint from Hainmueller and Hopkins (2015), the top levels are highly specific.

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_rank.png" title="Random-forest level importance, Hainmueller and Hopkins (2015) immigration conjoint" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

### Reading MDA in plain language

MDA, or Mean Decrease in Accuracy, answers a simple question. If we secretly scrambled this one attribute level so it carried no information, how much worse would the model's predictions get?

- **A high MDA** means the model leans heavily on that level. If you remove it, predictions fall apart. In behavioral terms, respondents are using that level to make their choice, and flipping it changes who gets picked.
- **A low MDA** means the model barely notices when that level is shuffled. Predictions stay almost as accurate without it. That is the signature of a level respondents are effectively ignoring.
- **An MDA close to zero** for an entire attribute (across all of its levels) is a strong cue for non-attendance. The design varied that attribute, but it did not enter the decision.

Treat MDA as a behavioral footprint of which levels actually moved choices, alongside the AMCE coefficients (Breiman 2001). Read alongside the tree, the highest-MDA levels are exactly the ones you expect to see near the root, the red flags that gate the decision.

### The decision tree, red-flag avoidance made visible

The decision tree reveals the hierarchical elimination structure. The root split is the gatekeeper attribute, and deeper splits are conditional on earlier ones.

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_tree.png" title="Decision tree, Hainmueller and Hopkins (2015)" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

Read the tree as the decision process a respondent runs when they look at an immigrant profile. Each split is a red flag being checked, and the tree captures the order in which those red flags are screened.

1. **Root, "no plans to look for work."** This is the gatekeeper. Of the 2,000 profiles, 24% carry this level, and for those the chosen-rate collapses to 0.35. The branch terminates immediately. Respondents do not bother weighing the rest of the profile. A single attribute level has eliminated the alternative. This is the "elimination by aspects" step in the sense of Tversky (1972), where a red flag triggers a pass on the profile.
2. **Second screen, prior unauthorized entry.** Among the 76% of profiles that survive the first screen, the next thing respondents check is whether the immigrant once entered the country without authorization. That flag drops the chosen-rate from 0.58 to 0.42. Another red flag, another elimination round.
3. **Third screen, no college degree.** Profiles that clear both prior screens are then evaluated on education. A college degree raises the chosen-rate to 0.71. Absence of one drops it to roughly 0.56 and triggers a further conditional check.
4. **Within-branch qualifiers, language.** Only after the major red flags are cleared does a softer attribute, language ability, start to matter. Even then it does so only inside specific branches (fluent English among prior-violators, interpreter use among the non-degreed).

The tree shows something different from "which attributes have a big AMCE." It shows the **sequence of red-flag checks** respondents actually run. Levels that never appear in the tree (country of origin, gender, profession, application reason, job experience) are levels respondents do not screen on at all in this design. The design varied them, but they did not make it into the decision.

This is the diagnostic value. AMCEs and marginal means tell you the average pull of each level by design and stay deliberately agnostic about how the choice was reached. `cjdiag` is positioned around the complementary task, **decision-structure elicitation**. It recovers the hierarchy respondents impose on the choice task, alongside the marginal pull of each level that AMCEs already give you.

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

## References

- Breiman, L. (2001). [Random Forests](https://doi.org/10.1023/A:1010933404324). *Machine Learning*, 45(1), 5–32.
- Dill, J., Howlett, M., and Müller-Crepon, C. (2024). [At Any Cost: How Ukrainians Think about Self-Defense Against Russia](https://doi.org/10.1111/ajps.12832). *American Journal of Political Science*, 68(4), 1460–1478.
- Hainmueller, J., Hopkins, D. J., and Yamamoto, T. (2014). [Causal Inference in Conjoint Analysis: Understanding Multidimensional Choices via Stated Preference Experiments](https://doi.org/10.1093/pan/mpt024). *Political Analysis*, 22(1), 1–30.
- Hainmueller, J., and Hopkins, D. J. (2015). [The Hidden American Immigration Consensus: A Conjoint Analysis of Attitudes toward Immigrants](https://doi.org/10.1111/ajps.12138). *American Journal of Political Science*, 59(3), 529–548.
- Ham, D. W., Imai, K., and Janson, L. (2024). [Using Machine Learning to Test Causal Hypotheses in Conjoint Analysis](https://doi.org/10.1017/pan.2023.41). *Political Analysis*, 32, 329–344.
- Jenke, L., Bansak, K., Hainmueller, J., and Hangartner, D. (2021). [Using Eye-Tracking to Understand Decision-Making in Conjoint Experiments](https://doi.org/10.1017/pan.2020.11). *Political Analysis*, 29(1), 75–101.
- Tversky, A. (1972). [Elimination by Aspects: A Theory of Choice](https://doi.org/10.1037/h0032955). *Psychological Review*, 79(4), 281–299.
- Tversky, A., and Sattath, S. (1979). [Preference Trees](https://doi.org/10.1037/0033-295X.86.6.542). *Psychological Review*, 86(6), 542–573.

## Funding

Development of `cjdiag` is supported by the European Research Council (ERC) under the European Union's Horizon Europe research and innovation programme, project AGAPP "Algorithmic Governance — A Public Perspective" (ERC Starting Grant, grant agreement No. 101116772, PI: Prof. Daria Gritsenko).
