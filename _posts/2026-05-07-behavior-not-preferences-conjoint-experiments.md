---
layout: post
title: "Beyond Preferences. How Respondents Actually Decide in Conjoint Experiments"
author: David Karpa
date: 2026-05-07 06:00:00
description: AMCEs are valid by design and stay deliberately silent on the decision process. My recent methods work is about that decision process, and the cjdiag R package is the tool I built for it.
tags: research methods conjoint behavior cjdiag eba
categories: research
thumbnail: assets/img/cjdiag_tree.png
giscus_comments: false
related_posts: false
---

My recent methods work has been pulling in one direction. I want to know how respondents in conjoint experiments actually decide, not just what they prefer on average. Conjoints produce beautifully clean estimates of average marginal component effects (AMCEs) and marginal means, and political scientists rely on those quantities every day. Nothing in what follows says those estimates are wrong. They aren't. The AMCE is process-agnostic by design — Hainmueller, Hopkins, and Yamamoto (2014) are explicit about this — and that agnosticism is precisely what gives the estimator its robustness. The flip side is that, by construction, an AMCE tells you nothing about how the respondent in front of you got from two profiles to the one they ticked. That gap is exactly where I've been working, and `cjdiag` is the R package I built to fill it.

## A car-buying intuition

Think about the last time you actually bought a car, or even seriously considered it. The textbook microeconomic story is that you formed a utility function over price, fuel economy, brand, color, mileage, accident history, and so on, weighted each attribute, summed up the contributions for every car on offer, and picked the one with the highest score. That is not how anyone shops for a car. What people actually do looks more like a sequence of cuts:

1. **Fix a price category.** Anything above your budget is invisible. A €40,000 SUV doesn't enter the comparison even if it's the best car on the lot, because you never look at it. The price band is a hard gate, not one factor among many.
2. **Avoid red flags.** Inside the budget, certain features are deal-breakers and trigger an immediate pass. Heavy accident history. Wrong fuel type for your commute. A color you simply will not be seen in. Each of these eliminates the alternative on its own; nothing else about the car gets weighed against them.
3. **Compare what survives.** Only after the screens have done their work do mileage, equipment, and the small stuff actually get traded off against each other.

Tversky (1972) called this _elimination by aspects_. Picture the snack aisle at a kiosk: you're mildly hungry, mildly cheap, allergic to peanuts. You don't score every item out of ten. You throw out everything with peanuts, then everything over €2.50, and from whatever is left you grab what looks tastiest. Each step locks onto one _aspect_ — peanut content, price, taste — and eliminates everything that fails it. Tversky showed that this informal procedure has a clean probabilistic structure: the chance that you screen on a given aspect first is proportional to how much you care about it. Conjoint respondents do roughly the same thing when they look at two profiles, and `cjdiag` is what I use to recover the order of their screens.

## Why this matters for survey experiments

A conjoint task asks the respondent to pick the better of two profiles described by 5–10 attributes, and the standard analysis treats that answer as if they had quietly added up partial utilities behind the scenes. The AMCE answers one question very well: on average, how much does flipping this one level shift the chosen-rate? It is silent on a different one by design: did respondents actually weigh this attribute, or did they settle the choice on something else and never look at it?

When respondents in an immigration conjoint see "no plans to look for work," they don't down-weight that profile. They eliminate it — on average. The other six attributes barely enter the comparison for that pair. The AMCE will still recover an average effect — it averages across respondents who behaved that way and respondents who didn't, across pairs where the level appeared and pairs where it didn't — and that average is correct on its own terms. It just doesn't tell you how the choice was actually reached.

That has three concrete consequences for survey experiments. The first is that **non-attendance is invisible to AMCEs**. If half your respondents ignore an attribute entirely, the AMCE for that attribute averages real preferences with pure noise. The estimate comes out small, and a reader will naturally interpret it as a weak preference, when the actual story may be that many respondents never looked at the attribute at all. Both stories are consistent with the same AMCE, and the AMCE alone can't separate them. The second is that **the hierarchy is invisible to marginal means**. Marginal means tell you the chosen-rate at each level, but they can't tell you that one level acted as a gate while every other attribute was a tiebreaker among the survivors. Two designs with identical marginal means can imply very different decision processes. The third is that **external validity depends on the process**. Whether a conjoint result transports to real-world behavior depends on whether the real-world choice is also dominated by the same red flags. If respondents in a survey screen on price and ignore fuel type while real shoppers do the reverse, AMCEs alone won't catch the mismatch.

## What cjdiag does about it

The [`cjdiag`](/projects/cjdiag/) package is built around exactly this distinction. It runs alongside `cjoint` or `cregg` rather than replacing them — keep your AMCEs and add a second layer that recovers the decision structure underneath them.

The clearest illustration is the decision tree. Fit on the bundled Hainmueller and Hopkins (2015) immigration data, the tree reads almost literally as a sequence of red-flag screens.

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_tree.png" title="cjdiag decision tree, Hainmueller and Hopkins (2015) immigration conjoint" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

The root split is **"no plans to look for work."** A profile carrying that level is gone: predicted chosen-rate 0.35, branch terminates, no further screens. That is the conjoint analogue of "above the price cap" in the car example — a hard gate that ends the comparison before any other attribute is consulted. Among the profiles that survive the first screen, the next thing respondents check is **prior unauthorized entry**, which drops chosen-rate from 0.58 to 0.42. Profiles that clear that screen are then evaluated on **education**, where a college degree pushes chosen-rate to 0.71. Only deeper in the tree do softer attributes like **language ability** start to act as tiebreakers within particular branches.

What's striking is what's missing. Country of origin, gender, profession, application reason, job experience — none of these enter the tree at all. The design varied them, but in this analysis they didn't reach the threshold to gate any decision. AMCEs will still produce coefficients for each of them, and those coefficients are perfectly valid AMCEs. They just don't tell you that the choice was effectively settled before respondents got to the attributes those coefficients describe.

So instead of asking what the average pull of each level is, the tree asks what the **order of the screens** is. The first question is about preferences. The second is about behavior. They are complementary, not competing.

One caveat. A single decision tree is, of course, heavily overfit to the particular sample it was grown on, which is why `cjdiag` actually estimates a random forest in the background, growing hundreds of trees and learning from each. The tree picture above is the most readable summary of the structure, but the level-importance numbers and the gatekeeper diagnostics come from the forest, not from this one tree.

## What I am building toward

cjdiag is one piece of a broader project to take the behavioral content of conjoint data seriously. The package exposes five complementary methods through `cj_fit()`:

- `forest` recovers which levels actually move predictions, via Mean Decrease in Accuracy (Breiman 2001).
- `tree` shows the hierarchy of screens, as above.
- `crt` tests which levels survive a strict signal-vs-noise penalty (Ham, Imai and Janson 2024).
- `nmm` recovers the order in which levels settle choices (Dill, Howlett and Müller-Crepon 2024).
- `marginal_r2` reports how much of an individual respondent's choice variance one attribute explains on its own (Jenke et al. 2021).

Each is a different angle on the same underlying claim: the choice was produced by a process, that process was hierarchical, and the hierarchy is recoverable from the data we already collect. There is independent validation for this from eye-tracking. The levels respondents fixate on first in a conjoint task (Jenke et al. 2021) are the same ones the tree puts at the root.

A steep hierarchy is not a sign of a broken design. It is evidence that the design successfully activated a strong, structured preference, and surfacing that structure is what `cjdiag` is for. The car shopper who refuses to look at anything above their budget isn't being irrational. They're being efficient. When we analyze their choices, our job is to notice that the budget was the gate and to report it that way, alongside the AMCEs that quantify the average pull of everything that survived it.

---

_The `cjdiag` R package is available at [github.com/dkarpa/cjdiag](https://github.com/dkarpa/cjdiag) with documentation at [dkarpa.github.io/cjdiag](https://dkarpa.github.io/cjdiag/). Development is supported by the ERC project AGAPP (PI: Prof. Daria Gritsenko)._

## References

- Breiman, L. (2001). [Random Forests](https://doi.org/10.1023/A:1010933404324). _Machine Learning_, 45(1), 5–32.
- Dill, J., Howlett, M., and Müller-Crepon, C. (2024). [At Any Cost: How Ukrainians Think about Self-Defense Against Russia](https://doi.org/10.1111/ajps.12832). _American Journal of Political Science_, 68(4), 1460–1478.
- Hainmueller, J., Hopkins, D. J., and Yamamoto, T. (2014). [Causal Inference in Conjoint Analysis: Understanding Multidimensional Choices via Stated Preference Experiments](https://doi.org/10.1093/pan/mpt024). _Political Analysis_, 22(1), 1–30.
- Hainmueller, J., and Hopkins, D. J. (2015). [The Hidden American Immigration Consensus: A Conjoint Analysis of Attitudes toward Immigrants](https://doi.org/10.1111/ajps.12138). _American Journal of Political Science_, 59(3), 529–548.
- Ham, D. W., Imai, K., and Janson, L. (2024). [Using Machine Learning to Test Causal Hypotheses in Conjoint Analysis](https://doi.org/10.1017/pan.2023.41). _Political Analysis_, 32, 329–344.
- Jenke, L., Bansak, K., Hainmueller, J., and Hangartner, D. (2021). [Using Eye-Tracking to Understand Decision-Making in Conjoint Experiments](https://doi.org/10.1017/pan.2020.11). _Political Analysis_, 29(1), 75–101.
- Tversky, A. (1972). [Elimination by Aspects: A Theory of Choice](https://doi.org/10.1037/h0032955). _Psychological Review_, 79(4), 281–299.
- Tversky, A., and Sattath, S. (1979). [Preference Trees](https://doi.org/10.1037/0033-295X.86.6.542). _Psychological Review_, 86(6), 542–573.
