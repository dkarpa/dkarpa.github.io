---
layout: post
title: "Which Attributes Do Conjoint Respondents Actually Use? A Diagnostic"
author: David Karpa
date: 2026-05-07 06:00:00
description: AMCEs are valid by design and stay deliberately silent on which attributes respondents used. cjdiag reads a population-level attendance signal off the same choice data, validated against eye-tracking.
tags: research methods conjoint behavior cjdiag eba
categories: research
thumbnail: assets/img/cjdiag_rank.png
giscus_comments: false
related_posts: false
published: false
---

> **Draft — taken offline for revision.** This is a working draft kept in the repo while the framing is reworked. It is not currently rendered on the site (`published: false` in the front matter).

Conjoints produce clean estimates of average marginal component effects (AMCEs) and marginal means, and political scientists rely on those quantities every day. The AMCE is a precise, randomization-identified average effect, and Hainmueller, Hopkins, and Yamamoto (2014) are explicit that it makes no behavioural assumption. `cjdiag` adds a separate, descriptive quantity to the same data: for any given conjoint, how concentrated is the predictive signal across attribute levels — that is, do some levels carry most of the discriminating information in the choice data while others carry essentially none? This post is about what that descriptive reading buys you, and why it took a new estimator to make it available on standard 5–10-task conjoints rather than only on the long ones.

## A simple intuition

Picture the snack aisle at a kiosk. You're mildly hungry, mildly cheap, allergic to peanuts. You don't score every item out of ten. You toss out anything with peanuts, then anything over €2.50, then from whatever is left you grab what looks tastiest. Tversky (1972) wrote down one formal version of this kind of decision and called it _elimination by aspects_. Whether respondents in a conjoint do anything literally like that is not something choice data alone can tell us — Tversky and Sattath (1979) showed that several different rules can produce indistinguishable choice probabilities, and that result still holds. But these decisions do leave a recognizable signature in aggregate data: a few attributes carry most of the discriminating information, others carry very little. That signature is what `cjdiag` summarises. It does not identify the rule.

## What cjdiag adds

The AMCE answers one question precisely: on average across respondents, how much does flipping this level shift the chosen-rate? That is exactly the quantity researchers want when reporting which levels move choice in the population. `cjdiag` adds a second, descriptive question to the same data: in this dataset, how concentrated is the predictive signal across levels? If a small number of levels carry most of the discriminating information and several others carry very little, that is a fact about the data worth reporting alongside the AMCEs. It is not a claim that respondents executed any particular rule — choice data, as noted, do not pin down the rule — but it is a useful summary of where the signal sits.

There is one place where this descriptive summary connects to a long-running concern in the literature. Tversky (1972) raised the question of whether respondents in multi-attribute choice are using all the attributes at all, or whether some are effectively skipped. The choice-data answer is: we cannot identify "true non-attendance" cleanly from a "very small preference," because both produce the same near-zero signal. Under the kind of decision rule Tversky described, these two cases are also substantively the same — the attribute is not driving the choice either way. So the working question becomes the simpler one: does the level carry meaningful signal in the population? That is the question `cjdiag` is designed to answer.

## Why between-subject

There are existing tools for attribute non-attendance — latent-class ANA models, nested marginal means, per-respondent $R^2$ from ratings — and they recover respondent-specific attention weights. The price is that they need many tasks per respondent, typically 20 or more, before within-subject estimation has enough leverage. That cognitive load almost certainly changes how respondents decide: fixation coverage drops over tasks, satisficing rises with task load. What within-subject ANA tools identify is plausibly the non-attendance induced by long task batteries, not the attendance that operates in a standard 5–10-task conjoint. The published literature sits at 5–10 tasks. `cjdiag` reads attendance off between-subject variation in choice plus the design, so it applies to conjoints of any length — including most of the ones we already have data for.

## The HierNet / CRT estimator

The estimator the rest of this post leads with is HierNet, the L1-regularised hierarchy-constrained logistic regression of Bien and Tibshirani (2013), wrapped in the conditional randomisation test of Ham, Imai and Janson (2024). It is a between-subject estimator that asks, for each attribute level, the simplest possible version of the descriptive question: does the coefficient survive a strict penalty on model complexity? Levels whose coefficients get shrunk to exactly zero at modest penalty levels are levels for which the choice data, after accounting for everything else, carry no meaningful population-level signal.

The output is a regularisation path: as the penalty $\lambda$ increases, more coefficients get shrunk to zero, and the order in which they drop out is the descriptive ranking we want. The level that survives the longest is the one the choice data most insist on; the levels that drop out first are the ones the data are silent about. The path is read at the cross-validated $\lambda^{\!*}$ for the binary attendance flag, and across the whole path for a continuous robustness ranking.

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_rank.png" title="cjdiag CRT survival ranking, Hainmueller and Hopkins (2015) immigration conjoint" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

Two things to read off the picture. First, the ordering: a small subset of levels carries the bulk of the discriminating information in the choice data. Second, the tail: several levels drop out of the path almost immediately, which is the choice-data fingerprint of either non-attendance or a preference too small for the data to resolve — the two being substantively the same here.

Why HierNet specifically. We want the answer to "is this level carrying signal?" to be robust to small changes in specification — random forest importance can be noisy at the bottom of the ranking, and lasso-style penalisation gives a cleaner zero / non-zero readout. The hierarchy constraint (an interaction can only enter if both main effects do) is what lets the same machinery report interaction-level attendance without losing identifiability when many parameters are zero. None of this requires more than a handful of tasks per respondent.

## A second view: the decision tree

The same data fit with a decision tree gives a more readable picture of where the signal sits, at the cost of being a single-sample summary.

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_tree.png" title="cjdiag decision tree, Hainmueller and Hopkins (2015) immigration conjoint" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

The root split is **"no plans to look for work."** A profile carrying that level is rarely chosen on average: predicted chosen-rate 0.35, branch terminates, no further splits in this subtree. Among the profiles that clear that split, the next variable the tree picks up is **prior unauthorized entry**, dropping chosen-rate from 0.58 to 0.42. Profiles that clear that are then split on **education**, where a college degree pushes chosen-rate to 0.71. Only deeper in the tree do softer attributes like **language ability** appear.

What's striking is what's missing. Country of origin, gender, profession, application reason, job experience — none of these enter the tree at all. The design varied them, but they did not carry enough predictive signal to be selected as a split. AMCEs still estimate coefficients for each, and those coefficients are perfectly valid AMCEs. The tree just adds a second descriptive fact: in this dataset, those attributes contributed little to the population-level predictive signal — the same pattern the CRT path shows from a different angle.

One caveat: a single decision tree is heavily overfit to its sample, which is why `cjdiag` also fits a random forest in the background. The tree picture is the most readable summary; the importance numbers come from the forest.

## The other methods

The package exposes five methods through `cj_fit()`, each summarising the predictive-signal distribution in a slightly different way:

- `crt` reports which levels retain non-zero coefficients along the HierNet L1-regularised path (Ham, Imai and Janson 2024). The lead estimator above.
- `forest` summarises how much each level contributes to out-of-sample predictive accuracy, via Mean Decrease in Accuracy (Breiman 2001).
- `tree` displays the order in which levels split the data, as above.
- `nmm` reports the order in which levels separate choices in a nested aggregation (Dill, Howlett and Müller-Crepon 2024).
- `marginal_r2` reports per-respondent $R^2$ from regressing choice on each attribute alone (Jenke et al. 2021).

Each method reads attendance from a different angle. Where they agree, the population-level reading is robust to learner choice; where they disagree, the level deserves a closer look. There is independent validation from eye-tracking: in the Jenke et al. (2021) data, the levels respondents fixate on most are the ones the CRT and the forest both put at the top.

The aim is not to displace AMCEs but to put a second layer of description next to them: a population-level attendance reading, validated against eye-tracking on the datasets where both are available, that you can compute on conjoints we already have.

---

_The `cjdiag` R package is available at [github.com/dkarpa/cjdiag](https://github.com/dkarpa/cjdiag) with documentation at [dkarpa.github.io/cjdiag](https://dkarpa.github.io/cjdiag/). Development is supported by the ERC project AGAPP (PI: Prof. Daria Gritsenko)._

## References

- Bien, J., Taylor, J., and Tibshirani, R. (2013). [A Lasso for Hierarchical Interactions](https://doi.org/10.1214/13-AOS1096). _Annals of Statistics_, 41(3), 1111–1141.
- Breiman, L. (2001). [Random Forests](https://doi.org/10.1023/A:1010933404324). _Machine Learning_, 45(1), 5–32.
- Dill, J., Howlett, M., and Müller-Crepon, C. (2024). [At Any Cost: How Ukrainians Think about Self-Defense Against Russia](https://doi.org/10.1111/ajps.12832). _American Journal of Political Science_, 68(4), 1460–1478.
- Hainmueller, J., Hopkins, D. J., and Yamamoto, T. (2014). [Causal Inference in Conjoint Analysis: Understanding Multidimensional Choices via Stated Preference Experiments](https://doi.org/10.1093/pan/mpt024). _Political Analysis_, 22(1), 1–30.
- Hainmueller, J., and Hopkins, D. J. (2015). [The Hidden American Immigration Consensus: A Conjoint Analysis of Attitudes toward Immigrants](https://doi.org/10.1111/ajps.12138). _American Journal of Political Science_, 59(3), 529–548.
- Ham, D. W., Imai, K., and Janson, L. (2024). [Using Machine Learning to Test Causal Hypotheses in Conjoint Analysis](https://doi.org/10.1017/pan.2023.41). _Political Analysis_, 32, 329–344.
- Jenke, L., Bansak, K., Hainmueller, J., and Hangartner, D. (2021). [Using Eye-Tracking to Understand Decision-Making in Conjoint Experiments](https://doi.org/10.1017/pan.2020.11). _Political Analysis_, 29(1), 75–101.
- Tversky, A. (1972). [Elimination by Aspects: A Theory of Choice](https://doi.org/10.1037/h0032955). _Psychological Review_, 79(4), 281–299.
- Tversky, A., and Sattath, S. (1979). [Preference Trees](https://doi.org/10.1037/0033-295X.86.6.542). _Psychological Review_, 86(6), 542–573.
