---
layout: post
title: "A Formal Test for Attribute Attendance in Conjoint Experiments"
author: David Karpa
date: 2026-05-07 06:00:00
description: cjdiag adds a between-subject test for attribute attendance for standard conjoint experiment, with a p-value per level, validated against eye-tracking.
tags: research methods conjoint attendance cjdiag ana
categories: research
thumbnail: assets/img/cjdiag_crt_pval.png
giscus_comments: false
related_posts: false
published: true
---

Conjoint experiments randomise multi-attribute profiles and ask respondents to choose between them. The standard output is the average marginal component effect (AMCE), the average change in chosen-rate when one attribute level is flipped. The AMCE is identified by the design's randomisation, and Hainmueller, Hopkins, and Yamamoto (2014) are explicit that it rests on no behavioural assumption. It is the right quantity for the question it answers.

It does not answer a second question that often comes up in interpretation: did respondents attend to a given attribute at all? A level can carry a small AMCE because respondents weighed it and landed close to indifferent, or because a share of respondents never engaged with it. The AMCE averages over both cases and does not separate them. `cjdiag` adds a quantity that does: a between-subject test of whether an attribute level carries any (direct or interaction) population-level signal in the choice data, i.e., whether respondents on average attended to it. The output is a p-value per level, computed from the same choice data and the design. This post walks through that test and the package around it.

## Why attendance is worth testing

People simplify multi-attribute choices. Since Simon (1955), a long line of work has found that decision-makers under cognitive or time constraints do not weigh every attribute of every option. Think of the snack shelf at a kiosk: you are mildly hungry, mildly cheap, allergic to peanuts. You do not score each item out of ten. You drop anything with peanuts, drop anything over a price you have in mind, and pick from what is left. Some attributes settle the choice; others never get used.

Conjoint profiles are multi-attribute choices made quickly, so the same simplification is plausible there, and eye-tracking shows it directly. Jenke et al. (2021) tracked 122 respondents on an 11-attribute candidate conjoint and found they fixated only 45 to 71 percent of the attribute cells, with coverage tilted toward the more predictive attributes. Bansak and Jenke (2025) report that respondents process attributes largely one at a time rather than in concert.

The term for an attribute receiving no decision weight is attribute non-attendance (ANA), i.e., a respondent placing no weight on it. Choice data alone cannot identify which decision rule a respondent ran — Tversky and Sattath (1979) showed that several rules generate indistinguishable choice probabilities — but the narrower question, whether a level carried signal, is answerable from choice data, and that is the question `cjdiag` targets.

## The estimand: Average Effective Attendance

`cjdiag` estimates Average Effective Attendance (AEA). AEA is the average, across respondents, of how much each respondent's choice depends on a given attribute level. Each respondent contributes a reading on the unit interval; AEA is the population mean of that reading, the same averaging convention the AMCE uses (the _average_ marginal component effect).

The name states what it is. _Average_: a mean over respondents. _Effective_: the levels effectively attended, a set that for some respondents may be small. _Attendance_: whether the respondent placed any decision weight on the level.

AEA is descriptive, population-level, and identified from choice data plus the design's randomisation. It sits alongside the AMCE rather than replacing it. It is not a decision-rule model, and it is not a per-respondent attendance estimate.

## Why a between-subject estimator

Attendance is not a new target. Latent-class attendance models, nested marginal means (Dill, Howlett, and Müller-Crepon 2024), and per-respondent $R^2$ from ratings (Jenke et al. 2021) all recover attention, but they do it within-subject: they estimate respondent-specific weights, which needs many tasks per respondent, typically twenty or more.

That requirement carries a cost. Long task batteries change how respondents decide. Fixation coverage drops over the course of a session, and satisficing rises with task load. What a within-subject estimator picks up on a twenty-task battery is plausibly the non-attendance that the battery itself induced, not the attendance that operates in a conjoint of ordinary length. Most published conjoints run five to ten tasks, and the within-subject machinery does not apply to them.

AEA is estimated between-subject. It reads attendance off variation in choices across respondents combined with the design's randomisation, so it works at any task count, including the short conjoints that make up most of the published record.

## The estimator: an L1-penalised regularisation path

The estimator is a logistic regression of the choice outcome $Y$ on every attribute-level dummy in the profile $X$, with an $L_1$ penalty on the coefficients. The package uses HierNet, the hierarchy-constrained lasso of Bien, Taylor, and Tibshirani (2013), in the form Ham, Imai, and Janson (2024) brought to conjoint analysis:

$$
\hat\beta(\lambda) = \arg\min_\beta \Big\{ -\ell(\beta;\, Y, X) + \lambda \sum_j |\beta_j| \Big\}.
$$

The penalty $\lambda$ taxes the size of the coefficients. At $\lambda = 0$ every level's coefficient is estimated freely. As $\lambda$ rises, weak coefficients are driven to exactly zero, one at a time, until at a large enough $\lambda$ only the levels with the strongest choice signal remain. Each level therefore has a lifetime on the path: a range of $\lambda$ over which its coefficient stays non-zero. HierNet's hierarchy constraint, that an interaction term may enter only if both of its main effects do, means the same fit can register a level that matters through an interaction, not only one that matters on its own.

### Reading 1: λ-survival, a continuous ranking

For each attribute level $\ell$, define

$$
\mathrm{surv}(\ell) = \max\{\, \lambda : \hat\beta_\ell(\lambda) \neq 0 \,\},
$$

the largest penalty at which $\ell$ still has a non-zero coefficient. Normalised to the unit interval, $\mathrm{surv}(\ell)$ is the continuous AEA reading. A level that survives until the penalty is severe is one the choices depended on. A level shrunk to zero early carries no choice-discriminating signal beyond what the rest of the profile already explains.

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_crt_survival.png" title="CRT lambda-survival path, Hainmueller and Hopkins (2015) immigration conjoint" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

Each row is one of the 50 attribute levels in the Hainmueller and Hopkins (2015) immigration conjoint. Blue marks the values of $\lambda$ at which the level keeps a non-zero coefficient; grey marks where it has been shrunk to zero. "No plans to look for work" holds its coefficient across the full grid. The levels at the bottom drop out almost at once, which is the choice-data signature of non-attendance, or of a preference too small for the data to resolve. The two are the same here: either way the level is not moving choices.

### Reading 2: the CRT p-value, a formal test

The survival path orders the levels but does not, by itself, say where to draw the line between attended and not. The conditional randomization test (CRT) of Ham, Imai, and Janson (2024) draws it.

For a level $\ell$, the CRT asks whether $\ell$ contributes to choice in the population through any channel: a main effect, an interaction, or heterogeneity across respondents. It compares the HierNet test statistic computed on the observed data to the distribution of that statistic under $B$ resamples of $\ell$'s column, each drawn from the design's known randomisation while the rest of the profile is held fixed. Because a conjoint design fixes the distribution the levels were drawn from, those resamples are exact, and the resulting p-value is valid in finite samples, with no appeal to asymptotics and no assumption that the HierNet model is correctly specified.

The test statistic is computed at a single $\lambda$ fixed in advance by sparse-recovery theory, $\lambda = \sqrt{N \log p}$, with $N$ the number of choice tasks and $p$ the number of free coefficients. The value of $\lambda$ affects only the power of the test; the p-value stays valid for any choice of it. A level is declared attended when its p-value falls below 0.05.

This is the practical core of the package. For each attribute level in a conjoint of ordinary length, you get a hypothesis test with an interpretable p-value, computed from the choice data and the design alone. No extended task battery, no added design features, no eye-tracking equipment.

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_crt_pval.png" title="Marginal means and CRT attendance verdict by level, Hainmueller and Hopkins (2015)" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

The figure places the CRT verdict next to the marginal mean for all 50 levels. Each point and interval is a level's marginal mean, the raw chosen-rate of profiles carrying it, with a 95 percent interval; the colour marks whether the CRT flags the level as attended (here 36 of 50). The vertical line at 0.5 is indifference.

The marginal mean and the attendance verdict answer different questions, and the figure shows where they part. The marginal mean is an average chosen-rate; the CRT asks whether the level carries signal once the rest of the profile and respondent heterogeneity are accounted for. A level can sit close to 0.5 and still be attended, if it pushes choice in opposite directions across subgroups so the average washes out. A level can sit away from 0.5 and fail the test, if its deviation is small relative to sampling noise. Reading one off the other would mislabel both kinds of level.

## A model-free cross-check: the random forest

The penalised path is one estimator. `cjdiag` also fits a random forest as a second, non-parametric opinion on the same data. The forest ranks levels by Mean Decrease in Accuracy (MDA), i.e., how much the forest's out-of-sample predictions worsen when one level's values are randomly shuffled. A level with high MDA is one the forest leans on; a level near zero is one it barely uses.

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_rank.png" title="Random-forest level importance (MDA), Hainmueller and Hopkins (2015) immigration conjoint" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

MDA is noisier near the bottom of the ranking than the lasso's clean zero / non-zero readout, so `cjdiag` treats the CRT test as the attendance verdict and the forest as a robustness check. Where the two agree, the reading does not depend on the learner; where they diverge, the level is worth a closer look.

## A readable single-sample view: the decision tree

A single decision tree fit to the same data puts the structure in one picture.

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_tree.png" title="Decision tree, Hainmueller and Hopkins (2015) immigration conjoint" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

The root split is "no plans to look for work": profiles carrying that level have an average chosen-rate of 0.35, and the branch ends there. Among the profiles that clear it, the next split is prior unauthorized entry, which drops the chosen-rate from 0.58 to 0.42; profiles past that are split on education. Softer attributes such as language ability appear only deeper, inside specific branches. Country of origin, gender, profession, application reason, and job experience do not enter the tree at all. The design varied them, but in this dataset they did not carry enough signal to be selected as a split.

A single tree overfits its sample, so it is the readable summary rather than the estimate; the forest and the CRT path carry the numbers. The three views agree on the broad picture here, which is the reassuring case.

## Five methods, one interface

The package exposes five estimators through `cj_fit(formula, data, method)`:

- `crt` — the CRT attendance test and λ-survival path described above (Ham, Imai, and Janson 2024). The lead estimator.
- `forest` — random-forest level importance via Mean Decrease in Accuracy (Breiman 2001).
- `tree` — a single decision tree showing the order in which levels split the data.
- `nmm` — nested marginal means, the order in which levels separate choices under a nested aggregation (Dill, Howlett, and Müller-Crepon 2024).
- `marginal_r2` — per-respondent $R^2$ from regressing choice on each attribute separately (Jenke et al. 2021).

The CRT test is the one to report for an attendance verdict; the others read the same signal from different angles and serve as cross-checks.

## Validation against eye-tracking

The attendance verdict can be checked against where respondents actually look. On three eye-tracking conjoint datasets, Jenke et al. (2021) and the two scenarios in Bansak and Jenke (2025), the levels the CRT test flags as attended are, in rank, the levels respondents fixate most, with Spearman rank correlations between 0.22 and 0.33. The agreement is moderate and consistent across the three datasets. Eye-tracking measures where attention lands; the CRT test measures whether a level moved choices; the two are related but not identical, and a moderate correlation is about what one should expect. It is enough to support reading the test as an attendance diagnostic, and not so high as to suggest the two measure the same thing.

## Using it

Report AMCEs and marginal means for the marginal effects, as before. Add the CRT attendance test when the question is which levels carried signal in the choice data and which the data are silent about. It runs on conjoints of standard length, including ones already collected, and returns a p-value you can report per level.

---

_The `cjdiag` R package is available at [github.com/dkarpa/cjdiag](https://github.com/dkarpa/cjdiag) with documentation at [dkarpa.github.io/cjdiag](https://dkarpa.github.io/cjdiag/). Development is supported by the ERC project AGAPP (PI: Prof. Daria Gritsenko)._

## References

- Bansak, K., and Jenke, L. (2025). [Odd Profiles in Conjoint Experimental Designs: Effects on Survey-Taking Attention and Behavior](https://doi.org/10.1017/pan.2025.1). _Political Analysis_, 33, 315–338.
- Bien, J., Taylor, J., and Tibshirani, R. (2013). [A Lasso for Hierarchical Interactions](https://doi.org/10.1214/13-AOS1096). _The Annals of Statistics_, 41(3), 1111–1141.
- Breiman, L. (2001). [Random Forests](https://doi.org/10.1023/A:1010933404324). _Machine Learning_, 45(1), 5–32.
- Dill, J., Howlett, M., and Müller-Crepon, C. (2024). [At Any Cost: How Ukrainians Think about Self-Defense Against Russia](https://doi.org/10.1111/ajps.12832). _American Journal of Political Science_, 68(4), 1460–1478.
- Hainmueller, J., Hopkins, D. J., and Yamamoto, T. (2014). [Causal Inference in Conjoint Analysis: Understanding Multidimensional Choices via Stated Preference Experiments](https://doi.org/10.1093/pan/mpt024). _Political Analysis_, 22(1), 1–30.
- Hainmueller, J., and Hopkins, D. J. (2015). [The Hidden American Immigration Consensus: A Conjoint Analysis of Attitudes toward Immigrants](https://doi.org/10.1111/ajps.12138). _American Journal of Political Science_, 59(3), 529–548.
- Ham, D. W., Imai, K., and Janson, L. (2024). [Using Machine Learning to Test Causal Hypotheses in Conjoint Analysis](https://doi.org/10.1017/pan.2023.41). _Political Analysis_, 32, 329–344.
- Jenke, L., Bansak, K., Hainmueller, J., and Hangartner, D. (2021). [Using Eye-Tracking to Understand Decision-Making in Conjoint Experiments](https://doi.org/10.1017/pan.2020.11). _Political Analysis_, 29(1), 75–101.
- Simon, H. A. (1955). [A Behavioral Model of Rational Choice](https://doi.org/10.2307/1884852). _Quarterly Journal of Economics_, 69(1), 99–118.
- Tversky, A., and Sattath, S. (1979). [Preference Trees](https://doi.org/10.1037/0033-295X.86.6.542). _Psychological Review_, 86(6), 542–573.
