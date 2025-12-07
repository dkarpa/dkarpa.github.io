---
layout: page
title: Behavioral Realism in Conjoint Experiments
description: Attribute non-attendance, CRT-based diagnostics, and decision-tree approaches to understanding cognitive shortcuts in conjoint tasks.
img: /assets/img/ana_heatmap.jpg
importance: 1
category: work
related_publications: false
---

## Understanding Behavioral Realism in Conjoint Experiments

Conjoint experiments typically rely on marginal means or AMCEs to summarize preferences, implicitly assuming that respondents attend to all attributes and process information consistently. In reality, people often rely on cognitive shortcuts, selectively ignoring some attributes while focusing strongly on others. This project develops a behavioral-realist approach to conjoint analysis by incorporating *attribute non-attendance (ANA)* as a first-order feature of how individuals actually process information. I apply this framework to a preregistered survey experiment conducted in Finland (N = 2,040), where citizens evaluated automated travel-authorization systems in a politically charged context. The goal is to move beyond traditional estimands and show how partisan motivations shape which attributes respondents attend to and how they interpret them.

<div class="row justify-content-sm-center">
  <div class="col-sm-6 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/experimentflowchartpaper.jpg" title="Experiment structure" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm-6 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/CB_partisanship_dist.jpg" title="Distribution of partisanship" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Overview of the experimental flow (left) and the distribution of political attitudes across respondents (right).
</div>

## Estimating Attribute Non-Attendance Using CRT-Conjoint

To measure selective information use, I implement *conditional randomization tests* (CRTs) using the **CRTConjoint** package. Unlike standard choice models, CRTs test whether an attribute had *any* effect on choices—additive, interactive, or heterogeneous—without assuming a specific functional form. This makes attribute non-attendance empirically detectable rather than assumed. Across partisan groups and system-outcome conditions, the analysis reveals systematic patterns: core institutional attributes (e.g., responsibility, human involvement) are attended to consistently, while others (cost, appeal rights, data collection) are strategically ignored depending on respondents’ ideological priors. Together with open-ended responses, these diagnostics illustrate how motivated reasoning shapes selective attention in complex choice tasks.

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/ana.jpg" title="Attribute non-attendance heatmap" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/ana_heatmap.jpg" title="Mention probabilities in open-ended responses" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  CRT-based attribute non-attendance across partisan groups (left) and predicted mention probabilities from open-ended responses (right).
</div>

## Extending Behavioral Realism with Tree-Based Models

Attribute non-attendance is an important step toward behavioral realism, but tree-based methods can further uncover heterogeneous processing strategies. Decision trees and model-based recursive partitioning, for example, allow us to recover latent decision rules, interaction structures, and subgroup-specific heuristics that are obscured in pooled AMCEs. These methods identify respondent segments that systematically prioritize certain attributes, ignore others, or engage in threshold-based reasoning. In the next stage of this project, I extend the behavioral-realist toolkit by integrating decision-tree diagnostics with CRT-based ANA measures, producing a multidimensional account of how people actually reason through conjoint tasks—particularly in politically polarized contexts.

More content on the decision-tree component will be added soon.
