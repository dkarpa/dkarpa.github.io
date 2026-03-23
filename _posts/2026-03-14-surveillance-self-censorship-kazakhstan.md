---
layout: post
title: "Surveillance Silences the Informed: Evidence from Kazakhstan"
author: David Karpa
date: 2026-03-14 12:00:00
description: A survey experiment in Kazakhstan shows that surveillance-induced self-censorship is driven by an informed elite with access to international media.
tags: research surveillance self-censorship autocracy experiment
categories: research
thumbnail: assets/img/pobe_interaction_media.png
giscus_comments: false
related_posts: false
---

How do authoritarian regimes maintain their grip on public opinion beyond state-run media and the outright repression of journalists? One answer is surveillance. Not necessarily the kind that leads to arrests, but the kind that makes people think twice before speaking. When citizens know, or even suspect, that their online activity is monitored, they may stop expressing what they actually think. In a new paper, I test whether surveillance induces self-censorship in an autocracy and ask *who* self-censors.

There is a theoretical debate about how autocrats combine their tools. Some scholars argue that propaganda and information control *substitute* for repression: as regimes get better at shaping narratives, they need less coercion (Guriev and Treisman 2019, 2020). Others argue the opposite, that repression and information control *complement* each other (Gehlbach et al. 2022; Lamberova and Sonin 2023; Gohdes 2023). In this view, surveillance does not just gather intelligence. It signals repressive capacity, and that signal chills political speech. My experiment provides evidence for the complementarity account.

## The Experiment

I ran a survey experiment in Kazakhstan in November 2023 with 5,025 respondents. Kazakhstan is an ideal setting for studying digital surveillance. The government has deployed mass surveillance at the ISP level, VPNs are routinely cracked down on, and SIM cards must be registered with a government-issued ID. In 2019, the government became the first country to force its population to install a custom root certificate capable of decrypting content running through the country's largest internet service provider, targeting social media and communication services (Raman et al. 2020). The survey was fielded less than two years after the "Bloody January" protests of 2022, the largest mass mobilization in Kazakhstan's modern history, which left over 200 dead and thousands arrested.

Respondents were randomly assigned to one of three conditions. The control group received standard survey instructions. The surveillance group read a short text reminding them that the government can access their online activity. The privacy group read a text about encryption ensuring the confidentiality of their responses.

After the treatment, participants answered four questions. Three were politically sensitive: whether participating in protests for political change is justified, whether helping Russia avoid Western sanctions is justified, and whether Russia's "Special Military Operation" / invasion of Ukraine is justified. The framing of the last question was randomly assigned to balance demand effects. A fourth, non-sensitive question about working more than 50 hours a week served as a placebo. For each question, respondents could choose "Justified," "Not justified," or "Prefer not to answer." Choosing "prefer not to answer" is the measure of self-censorship.

## Surveillance Increases Self-Censorship, Privacy Does Not Recover It

The surveillance treatment increases the probability of answering "prefer not to answer" by 3.3 percentage points on average across the three sensitive questions (p<0.01). The placebo question about working hours shows no treatment effect, confirming that the surveillance prime specifically chills responses to politically sensitive topics rather than causing general survey anxiety.

The privacy treatment does not recover the effect. Telling people their responses are encrypted does not make them more willing to answer sensitive questions. The privacy condition yields a small, statistically insignificant effect of 0.3 percentage points. This asymmetry is consistent with loss aversion (Schmidt and Zank 2005), but it may also reflect something more specific to autocratic contexts. The surveillance treatment carried urgency and a concrete threat. The privacy treatment, promising encryption, was semantically closer to the neutral control condition and likely less credible in a country where the government has a track record of intercepting encrypted communications. For policies that aim to enhance political discourse, this means privacy-preserving technologies are no solution for increasing surveillance capabilities. They are costly, access is unevenly distributed, and they are simply not as effective because of loss aversion. The damage from surveillance salience is not easily reversed.

## Who Self-Censors? The Informed Elite

The average effect of 3.3 percentage points hides important heterogeneity. When I break the sample by media consumption, the entire surveillance effect is driven by citizens who consume international media, meaning news sources from outside Kazakhstan, whether Western, Russian, Turkish, or other non-Kazakh outlets.

This finding is grounded in a theoretical model by Gehlbach, Sonin, and Svolik (2022), who argue that autocracies strategically combine censorship, propaganda, and repression to shape citizen behavior. Their model predicts something counterintuitive. It is not the uninformed mass that self-censors most under surveillance but the informed citizen with direct access to alternative or foreign media. Informed citizens have more complete information, and they know that their information diverges from the official narrative. When they anticipate that expressing views contradicting the government line may trigger costly repression, the risk of punishment outweighs the expressive benefit of speaking up. The informed elite withholds negative information precisely because they understand the gap between what they know and what the government wants them to say.

The regression results confirm this. The interaction between the surveillance treatment and international media consumption is +5.3 percentage points (p<0.01). Meanwhile, the non-interacted surveillance effect turns insignificant. In plain language, the entire surveillance effect is concentrated among international media consumers. For citizens who rely only on domestic Kazakh media, the surveillance treatment has essentially no impact.

An interflex binning analysis following Hainmueller, Mummolo, and Xu (2019) sharpens the picture. Among those who do not consume international media, the surveillance treatment increases self-censorship by just 0.9 percentage points, statistically indistinguishable from zero. Among those who do consume international media, the effect jumps to 6.1 percentage points.

<div class="row mt-4 mb-4">
    <div class="col-12">
        {% include figure.liquid loading="eager" path="assets/img/pobe_interaction_media.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Predicted probability of answering "prefer not to answer" by treatment group and international media consumption. The surveillance effect is concentrated among citizens who consume international media. Based on avg_predictions() from the interaction model (Table 1, Column 3).
</div>

One detail makes this even more interesting. The overall association between international media consumption and self-censorship is actually negative (-12 percentage points, p<0.01). Foreign media consumers are generally *more* willing to answer questions. They are more politically engaged and more opinionated. But the surveillance treatment reverses this. The very citizens who are most willing to speak up are the ones who go quiet when surveillance becomes salient.

The paper also tests whether higher education drives the effect, following the same theoretical logic (the "elite" could be defined by education rather than media access). It does not. The education interaction effects are small and not statistically significant (2.2% for non-highly educated vs. 3.9% for highly educated, p>0.1). In Kazakhstan, what defines the informed elite is not formal education but access to a diverse media environment beyond the domestic information space.

## Why It Matters

Surveillance specifically silences the most informed citizens, those who have broken through the domestic information bubble and accessed alternative sources of news. This has concrete consequences.

Self-censoring citizens do not express their opinions on political issues, which contributes to the chilling of political discussions and to the depoliticization of individuals. Without knowledge of their peers' preferences on political issues, political opposition has difficulty organizing (King et al. 2017). New surveillance technologies can thus bolster the autocrat's power before unrest even forms, and can in turn be suppressed through facial recognition surveillance technology (Beraja et al. 2023). This is also relevant for public opinion research. If an informed elite systematically self-censors in response to surveillance, opinion polls in autocracies will overestimate pro-government sentiment and underestimate dissent (Corstange 2012; Frye et al. 2017; Robinson and Tannenberg 2019).

The results provide evidence that repression and information control complement rather than substitute for each other. Surveillance does not just gather intelligence. It signals repressive capacity, and that signal selectively silences the citizens who are best positioned to challenge the official narrative. The people who know the most say the least.

---

*This post is based on "Digital Surveillance and Self-Censorship in Autocracies: Evidence from a Survey Experiment in Kazakhstan" by David Karpa, published in [Political Behavior](https://link.springer.com/journal/11109).*
