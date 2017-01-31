+++
date = "2017-01-30T21:50:39-08:00"
title = "Actors are Redundant"

+++

First some necessary context. DREAD is a system of scoring security threads. 
The letters stand for:

- Damage
- Reproducibility
- Exploitability
- Affected Users
- Discoverability

We score each of these metrics on a numeric scale with lower numbers indicating
lower severity. A scale of 1 to 10 for each metric is common, though there is no
set requirement other than for a given scoring, all metrics should be scored on 
the same scale, because we average the metrics to provide a final, overall, 
DREAD score. This final score allows us to prioritize addressing our threats.

## Do we need actors?

I've seen a few posts on threat modeling recently and a number of them included
a concept of "actors." These are entities that may attempt to carry out an attack.
In doing so they bring up the idea that if an attack is only executable by, 
for example, a nation state level attacker, maybe you don't care about addressing
it. This is completely reasonable; it may be too expensive and time consuming
to address this level of attacker.

What I would like to posit here however is that with a well design DREAD scale
the concept of actors is redundant. The Exploitabiliy metric specifically 
addresses the level of skill and resources required to execute a specific
attack, and does so in a more empirical way.

While the other metrics are largely dependent on your specific project, 
Exploitability is more generic. I would go as far as to say Exploitability 
is even more objective when measure only against the skills and resources
required, without taking into account the specifics of your system.

To that end, I propose a 1 to 10 scale for Exploitability that attempts to
provide you with an objective, useful, and relatable "actors" that would be
able to carry at an exploit at each level. Starting from the bottom, they are:

1. A user acting in an unintended but non-malicious way.
2. A non-technical user acting in a malicious way.
3. A semi-technical user that can use but not modify available exploit tools.
4. A technical user that can tweak already available exploit tools.
5. A highly skilled engineer not an expert in the relevant domain.
6. A highly skilled domain expert for the relevant domain.
7. A single corporation.
8. Multiple corporations acting in collusion.
9. A single nation state.
10. Multiple nation states acting in collusion.

I would anticipate many of your threats will receive a 4-6 score based on
this scale and that's fine. The point of DREAD scoring is to prioritize your
threats against each other. The scoring only has to be internally consistent
to a given threat model.

On the other hand, without a well defined scale, we're vulnerable to scoring
abnormalities such as central tendency bias. You shouldn't be saving 10 or 1
for that super hard or super easy Exploit you haven't met yet. There should
be a justifiable reason for the score, and this can be provided with a 
well defined scale.

## Simplifying our model

With this scale we can now remove the concept of actors from a model.
Instead we appropriately score our threats and if our policy says we will
not spend resources addressing attacks that require nation state level
resources, we will not address threats with an Exploitability score of 9 or 10.