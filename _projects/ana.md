---
layout: page
title: Behavioral Realism in Conjoint Experiments
description: Attribute non-attendance, CRT-based diagnostics, and tree-based approaches for understanding cognitive shortcuts in conjoint tasks.
img: /assets/img/ana_heatmap.jpg
importance: 1
category: work
related_publications: false
---

## Understanding Behavioral Realism in Conjoint Experiments

Conjoint experiments typically rely on marginal means or AMCEs to summarize preferences, implicitly assuming that respondents attend to all attributes and process information consistently. In reality, people often rely on cognitive shortcuts, selectively ignoring some attributes while focusing strongly on others. This project develops a behavioral-realist approach to conjoint analysis by incorporating *attribute non-attendance (ANA)* as a first-order feature of how individuals actually process information. I apply this framework to a  survey experiment conducted in Finland (N = 2,040), where citizens evaluated automated travel-authorization systems in a politically charged context. The goal is to try behavioral beyond traditional estimands and show how partisan motivations shape which attributes respondents attend to and how they interpret them.

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

---

## Experimental Setup

We draw on a survey experiment fielded in October 2024 as part of the Citizen Barometer at the University of Helsinki.  
The sample consists of 2,040 Finnish adults, recruited to broadly match the national population. Respondents provided demographic background, attitudes toward automation, and their familiarity with automated border-control systems. Survey weights adjust the sample to reflect population benchmarks for region, language, age, gender, and education.

Because the project focuses on the political conditions under which people legitimize automated decision-making, we also measured political orientations. Instead of traditional party identification, we rely on a feeling-thermometer measure: respondents rated major parties from 0–10. The *distance* between evaluations of the left-wing (Vasemmistoliitto) and right-wing (Perussuomalaiset) parties provides a continuous indicator of political leaning. Following prior research, we divide respondents into **Left**, **Center**, and **Right** based on tertiles of this distribution.

<div class="row justify-content-sm-center">
  <div class="col-sm-8 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/CB_partisanship_dist.jpg" title="Distribution of partisanship" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Distribution of partisan orientations in the sample, measured as the absolute difference in evaluations of Finland's most polarized parties.
</div>

The core experiment began with a simple default-setting manipulation. Respondents were randomly assigned to one of three conditions:

- **Control** – no default mentioned  
- **Accept** – the system defaults to accepting all asylum seekers  
- **Reject** – the system defaults to rejecting all asylum seekers  

After this, respondents completed a **conjoint experiment** about automated border-control systems. Each system contained five attributes (e.g., data collected, human involvement, cost). In each task, respondents compared two systems, chose the one they preferred, and rated each system’s legitimacy. Each person completed three such tasks, giving six evaluations.

<table>
  <thead>
    <tr>
      <th><strong>Attribute</strong></th>
      <th><strong>Levels</strong></th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td><strong>1) Who makes entry decisions?</strong></td>
      <td>
        1a) Computer makes decisions without human involvement.<br>
        1b) Computer recommends, but a human makes the final decision.<br>
        1c) Computer makes decisions, but border guards can change the decision.
      </td>
    </tr>

    <tr>
      <td><strong>2) How much do applicants pay?</strong></td>
      <td>
        2a) Free to use.<br>
        2b) Costs EUR 7 per application.<br>
        2c) Costs EUR 100 per application.
      </td>
    </tr>

    <tr>
      <td><strong>3) Which kind of personal data will be collected?</strong></td>
      <td>
        3a) Collects ID, fingerprint, and face data.<br>
        3b) Collects ID, fingerprint, face, and health self-assessment data.<br>
        3c) Collects ID only.
      </td>
    </tr>

    <tr>
      <td><strong>4) Who is responsible when the system makes a mistake?</strong></td>
      <td>
        4a) The Finnish border guard (Rajavartiolaitos).<br>
        4b) Ministry of the Interior that developed the system.<br>
        4c) No one.
      </td>
    </tr>

    <tr>
      <td><strong>5) Is there a possibility to appeal?</strong></td>
      <td>
        5a) Appeals allowed, with a decision in 24 hours.<br>
        5b) Appeals allowed, with a decision in one week.<br>
        5c) The decision is final and cannot be appealed.
      </td>
    </tr>
  </tbody>
</table>


<div class="row justify-content-sm-center">
  <div class="col-sm-10 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/experimentflowchartpaper.jpg" title="Experiment Flow" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

Our analysis centers on how respondents *actually used* information while completing the conjoint tasks. Instead of assuming all attributes matter equally, we evaluate whether participants **attended to** each feature—and how this varied across political groups and treatment conditions.

---

## Estimating Attribute Non-Attendance with CRT-Conjoint

To detect selective information use, we apply the **Conditional Randomization Test (CRT)** implemented in the  
<a href="https://doi.org/10.1017/pan.2023.41" target="_blank"><strong>CRTConjoint</strong></a> package.

Unlike traditional conjoint analyses, CRTs test a simple question:  
**Did this attribute make *any* difference to respondents’ choices?**  

The test does not assume additivity or linearity and captures any possible effect—interactive, nonlinear, or subgroup-specific. If the CRT rejects the null hypothesis, the attribute was *attended to*. If not, it was effectively *ignored*. Patterns across partisan groups and treatment arms reveal systematic motivated reasoning: people attend to attributes differently depending on whether the system's outcome aligns with their political priors.

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/ana.jpg" title="Attribute non-attendance (CRT)" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/ana_heatmap.jpg" title="Mention probabilities in open-ended responses" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

<div class="caption">
  CRT-based ANA across partisan groups (left) and predicted mention probabilities from open-ended responses (right).
</div>

---

## Extending Behavioral Realism with Tree-Based Estimation

CRTs tell us *whether* an attribute mattered—tree-based models help us understand *how*.  
Using Bayesian Additive Regression Trees (BART) for conjoint experiments (`cjbart`), we estimate individual-level effects and compute **variable importance** by political group. This allows us to uncover hidden decision rules: nonlinearities, thresholds, and interactions that cannot be detected by AMCEs.

What is especially noteworthy is that tree-based diagnostics strongly mirror the CRT results. Across methods, we see:

- Consistent attention to institutional attributes (e.g., responsibility, human involvement)  
- Selective ignoring of cost, appeal rights, and data collection among some partisan groups  
- Parallel patterns across treatment arms  
- Robustness across statistical principles (randomization tests, ensemble methods, Bayesian trees)  

The next figure summarizes ANA across **three diagnostic procedures**—CRT, Random Forest MSE importance, and Random Forest node purity—illustrating how convergent the results are.

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/ana_heatmap_three_tests_R1000.jpg" title="ANA by CRT, RF-MSE, and RF-Purity" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

<div class="caption">
  Attribute non-attendance across partisan groups using three estimation strategies. Despite relying on different statistical foundations, all methods converge on the same substantive conclusion: respondents selectively attend to information in ways aligned with their political motivations.
</div>

Together, CRT diagnostics and tree-based models provide a multidimensional picture of how people actually process conjoint tasks—often relying on shortcuts, ignoring information, and attending selectively when an attribute reinforces or challenges political attitudes. More content on the decision-tree component will be added soon.
