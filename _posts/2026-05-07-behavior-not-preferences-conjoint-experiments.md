---
layout: post
title: "Beyond Preferences. How Respondents Actually Decide in Conjoint Experiments"
author: David Karpa
date: 2026-05-07 12:00:00
description: AMCEs are valid by design and stay deliberately silent on the decision process. My recent methods work is about that decision process, and the cjdiag R package is the tool I built for it.
tags: research methods conjoint behavior cjdiag eba
categories: research
thumbnail: assets/img/cjdiag_tree.png
giscus_comments: false
related_posts: false
---

A lot of my recent methods work has been pulling in one direction. I want to get at how respondents actually behave, not only what they prefer. Conjoint experiments are the workhorse design for measuring multidimensional preferences in political science. They produce clean estimates of average marginal component effects (AMCEs) and marginal means. The headline point of the work I am building is not that AMCEs are wrong or invalid. They are not. The AMCE estimand is process-agnostic by design. Hainmueller, Hopkins, and Yamamoto (2014) make this explicit. The estimator deliberately makes no behavioral assumption about how respondents reach a choice, which is exactly what makes it robust. The decision process is invisible to AMCEs by construction. That is a feature. But the decision process is what I am interested in, and `cjdiag` is the tool I built to recover it from the same data.

## A car-buying intuition

Think about the last time you actually bought, or seriously considered, a car. The textbook microeconomic story is that you formed a utility function over price, fuel economy, brand, color, mileage, accident history, and so on, weighted each attribute, summed up the contributions for every car on offer, and picked the one with the highest score.

That is not how anyone shops for a car.

What people actually do is closer to this.

1. **Fix a price category.** Anything above the budget is invisible. A €40,000 SUV does not enter the comparison even if it is the best car on the lot. The price band acts as a hard gate. Whatever else the car offers above that ceiling, the comparison ends right there.
2. **Avoid red flags.** Inside the budget, certain features are deal-breakers. Heavy accident history? Pass. Wrong fuel type for your commute? Pass. A color you simply will not be seen in? Pass. Each red flag eliminates the alternative outright, with no trade-off across features.
3. **Compare what survives.** Only after the screens have done their work do mileage, equipment, and the small stuff actually get weighed against each other.

This is **elimination by aspects** in the sense of Tversky (1972). Picture the snack aisle at a kiosk. You are mildly hungry, mildly cheap, and allergic to peanuts. Instead of scoring every item on a 1–10 scale, you throw out everything with peanuts, then everything above €2.50, and then from whatever survives you grab the one you actually like. Each step locks onto a single *aspect* (peanut content, price, taste) and eliminates anything that fails it. The choice is produced by a sequence of cuts. No partial scores get added up. Tversky's contribution was to show that this informal procedure has a clean probabilistic structure. The chance that you screen on any given aspect first is proportional to how much you care about it. Conjoint respondents do roughly the same thing, and `cjdiag` is built to recover the order of cuts.

## Why this matters for survey experiments

Conjoint experiments embed exactly this kind of choice (pick the better of two profiles described by 5–10 attributes). The AMCE answers one question very well. On average, how much does flipping this one level shift the chosen-rate? It is silent on a different question by design. Did respondents actually weigh this attribute, or did they settle the choice on something else and never look at this one?

When respondents in an immigration conjoint see "no plans to look for work," they do not down-weight that profile. They eliminate it. The other six attributes never enter the comparison for that pair. The AMCE still recovers an average effect, of course, and that average is correct on its own terms. It averages across respondents who behaved that way and respondents who did not, and across pairs where the level appeared and pairs where it did not. The number does what AMCEs are designed to do. It just does not tell us how the choice was reached.

This matters in three concrete ways.

- **Non-attendance is invisible to AMCEs.** If half your respondents ignore an attribute entirely, the AMCE on that attribute averages real preferences with pure noise. The estimate looks small, and a reader will naturally read it as a weak preference. What actually happened is that many respondents never considered the attribute. Both readings are consistent with the same AMCE.
- **Hierarchy is invisible to marginal means.** Marginal means tell you the chosen-rate at each level. They cannot tell you that one level acted as a gate while the rest were tiebreakers. Two designs with identical marginal means can imply very different decision processes.
- **External validity depends on the process.** Whether a conjoint result transports to real-world behavior depends on whether the real-world choice is also dominated by the same red flags. If respondents in a survey screen on price and ignore fuel type while real shoppers do the reverse, AMCEs alone will not catch the difference.

## What cjdiag does about it

The [`cjdiag`](/projects/cjdiag/) package is built around this distinction. It runs alongside `cjoint` or `cregg`. Keep your AMCEs and add a second layer that recovers the decision *structure*.

The decision tree is the most direct illustration. Fit on the bundled Hainmueller and Hopkins (2015) immigration data, the tree reads like a sequence of red flags.

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/cjdiag_tree.png" title="cjdiag decision tree, Hainmueller and Hopkins (2015) immigration conjoint" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

The root split is **"no plans to look for work."** A profile carrying that level is gone. Predicted chosen-rate 0.35, branch terminates, no further screens. This is the conjoint analogue of "above the price cap" in the car example, a hard gate that ends the comparison. Among profiles that survive, the next red flag respondents check is **prior unauthorized entry**, which drops chosen-rate from 0.58 to 0.42. Profiles clearing that screen are evaluated on **education**, where a college degree pushes chosen-rate to 0.71. Only deeper in the tree do softer attributes like **language ability** start to act as tiebreakers within branches.

What is striking is what is missing. Country of origin, gender, profession, application reason, job experience. None of these enter the tree at all. The design varied them, but respondents did not screen on them. AMCEs would still produce a coefficient for each, and those coefficients are valid AMCEs. The tree shows that the decision was effectively settled before those attributes were ever consulted, which is a separate fact about the process.

This is the diagnostic move. Instead of asking what the average pull of each level is, we ask what the **order of the screens** is. The first question is about preferences. The second is about behavior.

## What I am building toward

cjdiag is one piece of a broader project to take the behavioral content of conjoint data seriously. The package exposes five complementary methods through `cj_fit()`.

- `forest`, which levels actually move predictions (Mean Decrease in Accuracy, Breiman 2001)
- `tree`, the hierarchy of screens, as above
- `crt`, which levels survive a strict signal-vs-noise test (Ham, Imai and Janson 2024)
- `nmm`, the *order* in which levels settle choices (Dill, Howlett and Müller-Crepon 2024)
- `marginal_r2`, how much of each individual respondent's variance any single attribute explains (Jenke et al. 2021)

Each method is a different angle on the same underlying claim. The choice was produced by a process, that process was hierarchical, and the hierarchy is recoverable from the data we already collect. The companion paper, *Beyond the AMCE. Diagnosing Importance Hierarchies in Conjoint Experiments*, lays out the theory and validates it against eye-tracking data from Jenke et al. (2021), where the levels respondents fixate on first are also the ones the tree puts at the root.

A steep hierarchy does not signal a failure of the conjoint design. It shows that the design activated a strong, structured preference. Surfacing that structure is what `cjdiag` is for.

The car shopper who refuses to look at anything above their budget is being efficient. When we analyze their choices, our job is to notice that the budget was the gate and to report it that way, alongside the AMCEs that quantify the average pull of every other feature.

---

*The `cjdiag` R package is available at [github.com/dkarpa/cjdiag](https://github.com/dkarpa/cjdiag) with documentation at [dkarpa.github.io/cjdiag](https://dkarpa.github.io/cjdiag/). Development is supported by the ERC project AGAPP (PI: Prof. Daria Gritsenko).*

## References

- Breiman, L. (2001). [Random Forests](https://doi.org/10.1023/A:1010933404324). *Machine Learning*, 45(1), 5–32.
- Dill, J., Howlett, M., and Müller-Crepon, C. (2024). [At Any Cost: How Ukrainians Think about Self-Defense Against Russia](https://doi.org/10.1111/ajps.12832). *American Journal of Political Science*, 68(4), 1460–1478.
- Hainmueller, J., Hopkins, D. J., and Yamamoto, T. (2014). [Causal Inference in Conjoint Analysis: Understanding Multidimensional Choices via Stated Preference Experiments](https://doi.org/10.1093/pan/mpt024). *Political Analysis*, 22(1), 1–30.
- Hainmueller, J., and Hopkins, D. J. (2015). [The Hidden American Immigration Consensus: A Conjoint Analysis of Attitudes toward Immigrants](https://doi.org/10.1111/ajps.12138). *American Journal of Political Science*, 59(3), 529–548.
- Ham, D. W., Imai, K., and Janson, L. (2024). [Using Machine Learning to Test Causal Hypotheses in Conjoint Analysis](https://doi.org/10.1017/pan.2023.41). *Political Analysis*, 32, 329–344.
- Jenke, L., Bansak, K., Hainmueller, J., and Hangartner, D. (2021). [Using Eye-Tracking to Understand Decision-Making in Conjoint Experiments](https://doi.org/10.1017/pan.2020.11). *Political Analysis*, 29(1), 75–101.
- Tversky, A. (1972). [Elimination by Aspects: A Theory of Choice](https://doi.org/10.1037/h0032955). *Psychological Review*, 79(4), 281–299.
- Tversky, A., and Sattath, S. (1979). [Preference Trees](https://doi.org/10.1037/0033-295X.86.6.542). *Psychological Review*, 86(6), 542–573.
