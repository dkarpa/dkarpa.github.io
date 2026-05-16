---
layout: page
title: cjdiag — Diagnostic Tools for Conjoint Survey Experiments
description: An R package that adds a between-subject test for attribute attendance to conjoint choice data. It complements AMCEs and is validated against eye-tracking.
img: /assets/img/cjdiag_logo.png
importance: 1
category: software
related_publications: false
---

## What cjdiag does

Standard conjoint analysis tools (`cjoint`, `cregg`) estimate Average Marginal Component Effects (AMCEs), the causal effect of changing a single attribute level. The AMCE is identified by the design's randomisation and, as Hainmueller, Hopkins, and Yamamoto (2014) note, rests on no behavioural assumption. It answers a clear question: on average, how much does flipping a level shift the chosen-rate.

It does not answer a second question that often comes up in interpretation: did respondents attend to a given attribute at all? A level can carry a small AMCE because respondents weighed it and were close to indifferent, or because a share of respondents never engaged with it. The AMCE averages over both cases.

**cjdiag** adds a quantity that separates them: Average Effective Attendance (AEA), a between-subject test of whether an attribute level carries population-level signal in the choice data. The output is a p-value per level. cjdiag is a complement to `cjoint` and `cregg`, not a replacement: run those for AMCEs and marginal means, then run cjdiag for the attendance reading on the same data.

It works at the level of individual attribute levels rather than aggregated attributes, because whether a level carries signal depends on the specific level the researcher chose ("no plans to look for work" rather than "Job Plans" as a whole).

## Five methods, one interface

All methods are accessed through a single function, `cj_fit(formula, data, method)`.

| Estimand                  | `method =`      | Question                                                         | When to use                                       |
| ------------------------- | --------------- | ---------------------------------------------------------------- | ------------------------------------------------- |
| **Attendance test**       | `"crt"`         | Which levels carry population-level signal? A p-value per level. | Report alongside AMCEs for an attendance verdict. |
| **Level importance**      | `"forest"`      | Which levels contribute most to predictive accuracy?             | A model-free cross-check on the CRT ranking.      |
| **Decision structure**    | `"tree"`        | How does a single tree order the levels into splits?             | A readable one-picture summary of structure.      |
| **Decision order**        | `"nmm"`         | In what order do levels separate choices?                        | When the decision order is of interest.           |
| **Individual attendance** | `"marginal_r2"` | How much does each respondent's choice depend on each attribute? | When you want per-respondent heterogeneity.       |

The `crt` method is the one to report for an attendance verdict; the others read the same signal from different angles and serve as cross-checks. The verdict has been validated against eye-tracking: the levels `crt` flags as attended track the levels respondents fixate most in the Jenke et al. (2021) data.

## The attendance test

`crt` fits an L1-penalised logistic regression (HierNet; Bien, Taylor, and Tibshirani 2013) of the choice on every attribute-level dummy, and reads two quantities off the regularisation path.

The **λ-survival path** orders the levels. As the penalty rises, weak coefficients are driven to zero one at a time; a level's survival point is the largest penalty at which its coefficient is still non-zero. A level shrunk to zero early carries no choice-discriminating signal beyond the rest of the profile.

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_crt_survival.png" title="CRT lambda-survival path, Hainmueller and Hopkins (2015) immigration conjoint" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

The **CRT p-value** turns the path into a test. The conditional randomization test of Ham, Imai, and Janson (2024) compares the fit on the observed data to its distribution under resamples of one level's column, drawn from the design's known randomisation while the rest of the profile is held fixed. Because the design fixes that randomisation, the p-value is valid in finite samples. A level is declared attended when its p-value falls below 0.05.

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_crt_pval.png" title="Marginal means and CRT attendance verdict by level, Hainmueller and Hopkins (2015)" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

The marginal mean and the attendance verdict answer different questions. The marginal mean is an average chosen-rate; the test asks whether the level carries signal once the rest of the profile and respondent heterogeneity are accounted for. The figure places the two side by side for all 50 levels of the Hainmueller and Hopkins (2015) immigration conjoint, and they part for several levels: a level near the 0.5 indifference line can still be attended, and a level away from it can fail the test.

## A model-free cross-check: the random forest

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_rank.png" title="Random-forest level importance, Hainmueller and Hopkins (2015) immigration conjoint" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

The `forest` method ranks levels by Mean Decrease in Accuracy (MDA): how much the forest's out-of-sample predictions worsen when one level's values are randomly shuffled. A level with high MDA is one the forest leans on; a level near zero is one it barely uses. MDA is noisier near the bottom of the ranking than the lasso's zero / non-zero readout, so cjdiag treats the CRT test as the attendance verdict and the forest as a robustness check. Where the two agree, the reading does not depend on the learner.

## A readable single-sample view: the decision tree

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_tree.png" title="Decision tree, Hainmueller and Hopkins (2015) immigration conjoint" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

The `tree` method fits a single decision tree, which puts the structure in one picture. On the immigration conjoint, the root split is "no plans to look for work"; profiles carrying that level have an average chosen-rate of 0.35. Further splits pick up prior unauthorized entry and education, while several attributes (country of origin, gender, profession, application reason, job experience) do not enter the tree at all. The design varied them, but in this dataset they did not carry enough signal to be selected as a split. A single tree overfits its sample, so it is the readable summary rather than the estimate; the forest and the CRT path carry the numbers.

## Installation

```r
# install.packages("pak")
pak::pak("dkarpa/cjdiag")
```

## Links

- **GitHub:** [github.com/dkarpa/cjdiag](https://github.com/dkarpa/cjdiag)
- **Documentation:** [dkarpa.github.io/cjdiag](https://dkarpa.github.io/cjdiag/)
- **Getting Started vignette:** [cjdiag.html](https://dkarpa.github.io/cjdiag/articles/cjdiag.html)
- **Method-specific vignettes:** [crt](https://dkarpa.github.io/cjdiag/articles/crt.html), [forest](https://dkarpa.github.io/cjdiag/articles/forest.html), [tree](https://dkarpa.github.io/cjdiag/articles/tree.html), [nmm](https://dkarpa.github.io/cjdiag/articles/nmm.html), [marginal_r2](https://dkarpa.github.io/cjdiag/articles/marginal_r2.html)

## References

- Bien, J., Taylor, J., and Tibshirani, R. (2013). [A Lasso for Hierarchical Interactions](https://doi.org/10.1214/13-AOS1096). _The Annals of Statistics_, 41(3), 1111–1141.
- Breiman, L. (2001). [Random Forests](https://doi.org/10.1023/A:1010933404324). _Machine Learning_, 45(1), 5–32.
- Dill, J., Howlett, M., and Müller-Crepon, C. (2024). [At Any Cost: How Ukrainians Think about Self-Defense Against Russia](https://doi.org/10.1111/ajps.12832). _American Journal of Political Science_, 68(4), 1460–1478.
- Hainmueller, J., Hopkins, D. J., and Yamamoto, T. (2014). [Causal Inference in Conjoint Analysis: Understanding Multidimensional Choices via Stated Preference Experiments](https://doi.org/10.1093/pan/mpt024). _Political Analysis_, 22(1), 1–30.
- Hainmueller, J., and Hopkins, D. J. (2015). [The Hidden American Immigration Consensus: A Conjoint Analysis of Attitudes toward Immigrants](https://doi.org/10.1111/ajps.12138). _American Journal of Political Science_, 59(3), 529–548.
- Ham, D. W., Imai, K., and Janson, L. (2024). [Using Machine Learning to Test Causal Hypotheses in Conjoint Analysis](https://doi.org/10.1017/pan.2023.41). _Political Analysis_, 32, 329–344.
- Jenke, L., Bansak, K., Hainmueller, J., and Hangartner, D. (2021). [Using Eye-Tracking to Understand Decision-Making in Conjoint Experiments](https://doi.org/10.1017/pan.2020.11). _Political Analysis_, 29(1), 75–101.

## Funding

Development of `cjdiag` is supported by the European Research Council (ERC) under the European Union's Horizon Europe research and innovation programme, project AGAPP "Algorithmic Governance — A Public Perspective" (ERC Starting Grant, grant agreement No. 101116772, PI: Prof. Daria Gritsenko).
