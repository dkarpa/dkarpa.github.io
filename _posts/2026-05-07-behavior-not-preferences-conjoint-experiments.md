---
layout: post
title: "A Formal Test for Attribute Attendance in Conjoint Experiments"
author: David Karpa
date: 2026-05-07 06:00:00
description: cjdiag adds a between-subject test for attribute attendance in standard conjoint experiments, with a p-value per level, validated against eye-tracking.
tags: research methods conjoint attendance cjdiag ana
categories: research
thumbnail: assets/img/cjdiag_crt_pval.png
giscus_comments: false
related_posts: false
published: true
---

Conjoint experiments randomise multi-attribute profiles and ask respondents to choose between them. The standard output is the average marginal component effect (AMCE), the average change in chosen-rate when one attribute level is flipped. Because the design's randomisation identifies the AMCE, it needs no model of how respondents choose, as Hainmueller, Hopkins, and Yamamoto (2014) stress. It is the right quantity for the question it answers.

It does not answer a second question that often comes up in interpretation: did respondents attend to a given attribute at all? A level can carry a small AMCE because respondents weighed it and landed close to indifferent, or because a share of respondents never engaged with it. The AMCE averages over both cases and does not separate them. The R package `cjdiag` adds a test aimed at this second question. It is a between-subject test of whether an attribute level carries any population-level signal in the choice data, direct or through interactions, i.e., whether respondents on average attended to it. The output is a p-value per level, computed from the same choice data and the design. This post walks through that test and the package around it.

## Why attendance is worth testing

People simplify multi-attribute choices. Since Simon (1955), a long line of work has found that decision-makers under cognitive or time constraints do not take every attribute of every option into consideration. Think of the snack shelf at a kiosk: you are mildly hungry, do not want to overspend and are allergic to peanuts. You do not score each item out of ten. You drop anything with peanuts, drop anything over a price you have in mind, and pick from what is left. Some attributes settle the choice; others never get used.

Conjoint tasks are multi-attribute choices made quickly, so the same simplification is plausible there, which eye-tracking shows directly. Jenke et al. (2021) tracked 122 respondents on an 11-attribute candidate conjoint and found they fixated between 45 and 71 percent of the attribute cells, with coverage tilted toward the more predictive attributes. Bansak and Jenke (2025) report that respondents process attributes largely one at a time rather than evaluating profiles as a whole. Respondents go through the attributes, compare levels and stop once they have enough information.

The term for a respondent placing no decision weight on an attribute is attribute non-attendance (ANA). ANA has a history in both marketing and transport research (Hess et al. 2013; Yegoryan, Guhl, and Klapper 2020; Yegoryan, Guhl, and Paetz 2023). I build on this literature and introduce a new estimand for conjoint experiments.

## Average Effective Attendance

`cjdiag` estimates Average Effective Attendance (AEA), the average across respondents of how much each respondent's choice depends on a given attribute level. Each respondent contributes a reading on the unit interval and AEA is the population mean of that reading, the same averaging convention the AMCE uses (the _average_ marginal component effect).

The name states what it is. _Average_ refers to the mean over respondents, _effective_ to the levels a respondent effectively attended (a set that may be small for some respondents) and _attendance_ to whether the respondent placed any decision weight on the level.

AEA is a descriptive, population-level quantity, identified from choice data plus the design's randomisation. It complements the AMCE and is neither a decision-rule model nor a per-respondent attendance estimate.

## Why a between-subject estimator

Existing attendance models work within-subject and need many tasks per respondent. Latent-class attendance models recover attendance by estimating respondent-specific weights, which typically takes twenty or more tasks.

That requirement carries a cost because long task batteries change how respondents decide. Fixation coverage drops over the course of a session and satisficing rises with task load. A within-subject estimator run on a twenty-task battery therefore plausibly picks up the non-attendance that the battery itself induced, not the attendance that operates in a conjoint of ordinary length. Most published conjoints run three to eight tasks, too few for the within-subject machinery.

AEA is estimated between-subject. It reads attendance off variation in choices across respondents combined with the design's randomisation, so it works at any task count, including the shorter conjoints that make up most of the published record.

## The estimator: an L1-penalised regularisation path

The estimator is a logistic regression of the choice outcome $Y$ on every attribute-level dummy in the profile $X$, with an $L_1$ penalty on the coefficients. The package uses HierNet, the hierarchy-constrained lasso of Bien, Taylor, and Tibshirani (2013), in the form Ham, Imai, and Janson (2024) brought to conjoint analysis:

$$
\hat\beta(\lambda) = \arg\min_\beta \Big\{ -\ell(\beta;\, Y, X) + \lambda \sum_j |\beta_j| \Big\}.
$$

The penalty $\lambda$ taxes the size of the coefficients. At $\lambda = 0$ every level's coefficient is estimated freely. As $\lambda$ rises, weak coefficients are driven to exactly zero, one at a time, until at a large enough $\lambda$ only the levels with the strongest choice signal remain. Each level therefore has a lifetime on the path, the range of $\lambda$ over which its coefficient stays non-zero. HierNet's hierarchy constraint, that an interaction term may enter only if both of its main effects do, means the same fit can also register a level that matters only through an interaction.

### λ-survival, a continuous ranking

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

Each row is one of the 50 attribute levels in the Hainmueller and Hopkins (2015) immigration conjoint. Dark cells mark the values of $\lambda$ at which the level keeps a non-zero coefficient; light cells mark where it has been shrunk to zero. "No plans to look for work" holds its coefficient across the full grid. The levels at the bottom drop out almost at once, which is the choice-data signature of non-attendance, or of a preference too small for the data to resolve. The choice data cannot tell the two apart, since in both cases the level does not move choices.

### The CRT p-value, a formal test

The survival path orders the levels but does not, by itself, say where to draw the line between attended and not. For that line I build on the conditional randomization test (CRT) of Ham, Imai, and Janson (2024).

For a level $\ell$, the CRT asks whether $\ell$ contributes to choice in the population through any channel: a main effect, an interaction, or heterogeneity across respondents. It compares the HierNet test statistic computed on the observed data to the distribution of that statistic under $B$ resamples of $\ell$'s column, each drawn from the design's known randomisation while the rest of the profile is held fixed. Because a conjoint design fixes the distribution the levels were drawn from, those resamples are exact, which makes the resulting p-value valid in finite samples, with no appeal to asymptotics and no assumption that the HierNet model is correctly specified. In other words, the researcher randomised the levels and therefore knows their distribution exactly. Under the null that $\ell$ does not matter, they could replace each profile's observed value of $\ell$ with a fresh draw from the same randomisation scheme and the choices $Y$ should look statistically the same. If the real data look weirder than the resamples, the level mattered.

The test statistic is computed at a single $\lambda$ fixed in advance by sparse-recovery theory, $\lambda = \sqrt{N \log p}$, with $N$ the number of choice tasks and $p$ the number of free coefficients. The value of $\lambda$ affects only the power of the test; the p-value stays valid for any choice of it. A level is declared attended when its p-value falls below 0.05. The p-values shown here come from `CRTConjoint::CRT_pval()`, the implementation of Ham, Imai, and Janson, run once per level with $B = 200$; `cj_fit(method = "crt")` returns the survival path.

For each attribute level in a conjoint of ordinary length, the result is a hypothesis test with an interpretable p-value, computed from the choice data and the design alone. You need neither more tasks nor a different design, so you can still estimate AMCEs and marginal means, compare a new study with earlier ones that used the same design, and read both next to the attendance verdict. The test also runs on previously published data:

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_crt_pval.png" title="Marginal means and CRT attendance verdict by level, Hainmueller and Hopkins (2015)" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

The figure places the CRT verdict next to the marginal mean for all 50 levels. Each point and interval is a level's marginal mean, the raw chosen-rate of profiles carrying it, with a 95 percent interval; black marks the levels the CRT flags as attended (36 of 50) and grey the rest. The vertical line at 0.5 is indifference.

The marginal mean and the attendance verdict answer different questions and the figure shows where they part. A level can sit close to 0.5 and still be attended if it pushes choice in opposite directions across subgroups so that the average washes out. Sudan as country of origin, for example, has a marginal mean of 0.48 with an interval that covers 0.5 and still passes the test. A level can also sit away from 0.5 and fail the test if its deviation is small relative to sampling noise (there are other reasons and a paper is on its way!). Financial analyst sits at 0.57 and is not flagged. Reading one off the other would mislabel both kinds of level.

## A non-parametric cross-check: the random forest

Besides the penalised path, `cjdiag` fits a random forest as a second, non-parametric opinion on the same data. The forest ranks levels by Mean Decrease in Accuracy (MDA), i.e., how much the forest's out-of-sample predictions worsen when one level's values are randomly shuffled. A level with high MDA is one the forest leans on; a level near zero is one it barely uses.

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_rank.png" title="Random-forest level importance (MDA), Hainmueller and Hopkins (2015) immigration conjoint" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

The figure shows the 20 levels with the highest MDA. The forest's five highest-ranked levels are also the five that survive longest on the penalised path, and across all 50 levels the two rankings correlate at 0.64 (Spearman).

## A readable single-sample view: the decision tree

A single decision tree fit to the same data puts the structure in one picture.

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_tree.png" title="Decision tree, Hainmueller and Hopkins (2015) immigration conjoint" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

The root split is "no plans to look for work". Profiles carrying that level have an average chosen-rate of 0.34 and the branch ends there. Among the profiles that clear it, the next split is prior unauthorised entry, which drops the chosen-rate from 0.59 to 0.42. Profiles past that are split on fluent English (0.67 with it, 0.56 without). Education, country of origin, gender, profession, application reason, and job experience do not enter the tree at all. The design varied them, but at the default complexity setting none of them carried enough signal to be selected as a split.

A single tree is a coarse summary of its sample, so it is the readable picture rather than the estimate; the forest and the survival path carry the numbers. The three views agree on the broad picture here, since the tree's three splits are among the four levels that the forest and the survival path both rank highest.

## Five methods, one interface

The package exposes five estimators through `cj_fit(formula, data, method)`:

- `crt`: the λ-survival path described above, the basis of the CRT attendance test (Ham, Imai, and Janson 2024) and the lead estimator.
- `forest`: random-forest level importance via Mean Decrease in Accuracy (Breiman 2001).
- `tree`: a single decision tree showing the order in which levels split the data.
- `nmm`: nested marginal means, the order in which levels separate choices under a nested aggregation (Dill, Howlett, and Müller-Crepon 2024).
- `marginal_r2`: per-respondent $R^2$ from regressing choice on each attribute separately (Jenke et al. 2021).

The CRT is the one to report for an attendance verdict; the others read the same signal from different angles and serve as cross-checks.

## Validation against eye-tracking

The attendance verdict agrees moderately with where respondents actually look. A paper on this validation is in progress; so far, on three eye-tracking conjoint datasets (Jenke et al. 2021 and the two scenarios in Bansak and Jenke 2025), levels with smaller CRT p-values are fixated more, with Spearman rank correlations between 0.22 and 0.33. Eye-tracking measures where attention lands, whereas the CRT measures whether a level moved choices, so a moderate correlation is about what one should expect. It is enough to support reading the test as an attendance diagnostic and not so high as to suggest the two measure the same thing.

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
- Hess, S., Stathopoulos, A., Campbell, D., O'Neill, V., and Caussade, S. (2013). [It's Not That I Don't Care, I Just Don't Care Very Much: Confounding between Attribute Non-Attendance and Taste Heterogeneity](https://doi.org/10.1007/s11116-012-9438-1). _Transportation_, 40(3), 583–607.
- Jenke, L., Bansak, K., Hainmueller, J., and Hangartner, D. (2021). [Using Eye-Tracking to Understand Decision-Making in Conjoint Experiments](https://doi.org/10.1017/pan.2020.11). _Political Analysis_, 29(1), 75–101.
- Simon, H. A. (1955). [A Behavioral Model of Rational Choice](https://doi.org/10.2307/1884852). _Quarterly Journal of Economics_, 69(1), 99–118.
- Yegoryan, N., Guhl, D., and Klapper, D. (2020). [Inferring Attribute Non-Attendance Using Eye Tracking in Choice-Based Conjoint Analysis](https://doi.org/10.1016/j.jbusres.2019.01.061). _Journal of Business Research_, 111, 290–304.
- Yegoryan, N., Guhl, D., and Paetz, F. (2023). [When Zeros Count: Confounding in Preference Heterogeneity and Attribute Non-attendance](https://ideas.repec.org/p/rco/dpaper/482.html). Rationality and Competition CRC TRR 190, Discussion Paper 482.
