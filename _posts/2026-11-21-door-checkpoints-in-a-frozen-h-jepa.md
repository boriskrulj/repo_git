---
layout: post
title: "Door checkpoints in a frozen H-JEPA: what worked and what didn't"
date: 2026-11-21
tags: [world-models, planning, jepa, lab-notebook]
description: "Do recurring states from successful episodes act as useful checkpoints for a hierarchical world model's planner? Results so far, with the limits."
excerpt: "Do recurring states from successful episodes act as useful checkpoints for a hierarchical world model's planner? Results so far, with the limits."
---

*Update, 24 November 2026: the project has moved on, and the current summary is in [Learning checkpoints from experience, and planning with them alone]({{ site.baseurl }}{% post_url 2026-11-24-learning-checkpoints-from-experience %}). This post is kept as the record of the earlier steps.*

*Status: ongoing lab notebook. One small world, three trained copies of one model. Results are reported with their limits, and I will update this post as new tests finish. Last update: 21 November 2026, with the replication on two more trained copies.*

**The short version.** In a four-room navigation task, I found states that successful episodes keep passing through (the doorways) using a frozen model's own embeddings, with door labels from the simulator to say which group is which door. On one trained copy of the model (seed 42), a planner nudged through those states reached the goal on 80 of 80 fresh trips, against 67 of 80 for the unmodified planner, and random anchors in the next room did no better than the unmodified planner.

**That did not replicate cleanly.** On a second trained copy (seed 43) the nudged planner did worse than the unmodified one (43 against 54 of 80), and on a third (seed 44) the unmodified planner already solved 75 of 80, so the test could not show a gain. The doorway groups themselves were found again on both copies. A later attempt to find the doors without any labels gave one positive result and then failed its own replication. So what stands is: the door structure is there on all three copies, the planning benefit was clear on one, and I do not yet know why.

## The question

H-JEPA (Zhang, Terver, Rabbat, LeCun, Balestriero; [arXiv:2610.06805](https://arxiv.org/abs/2610.06805)) trains a hierarchy of action-conditioned world models. The upper level predicts further ahead in its own latent space. Planning runs top-down: the upper level proposes a subgoal and the lower level plans towards it. The authors release their code at [kevinghst/H-JEPA](https://github.com/kevinghst/H-JEPA).

I took one of their frozen two-level models, trained for the four-room navigation task, and asked a narrow question:

> If I look at many *successful* episodes and ask which states recur, do those states work as useful checkpoints for the planner?

I never retrain the model. It still predicts outcomes and the planner still replans every few steps. The only change is a constraint on which candidate plans the planner prefers.

## The idea

In a four-room world, every trip between rooms squeezes through a doorway. Inside a room, successful episodes take many different paths. At the doorway they all look alike. A state that nearly every successful episode on the same route shares, and that few other states share, is a natural checkpoint.

## What I did

1. Cluster the upper-level embeddings of states from successful episodes (k-means, 16 clusters).
2. Keep the clusters that many *distinct episodes* of the same route visit. I count episodes, not frames, and leave out the last few states before the goal. Comparing against neighbouring doors keeps a cluster that merely sits inside a room from being mistaken for a door.
3. Check on held-out episodes that those clusters still recur on trips the clustering never saw.
4. Steer the planner. Rank candidate plans by how well they pass through the checkpoints in order, then by distance to the goal, and always execute the best plan.

## What came out

The test uses wall-blocked trips: one door is closed, so the direct route is blocked. I scored 80 fresh trips with a rule written down *before* the run: the agent gets within 16 px of the goal, having passed the true doors in order.

<figure class="chart">
<svg viewBox="0 0 640 200" role="img" aria-label="Trips reaching the goal out of 80: unmodified planner 67, doors given by the simulator 80, doors read from the embeddings 80." width="100%" style="max-width:640px">
  <g font-family="system-ui, sans-serif" font-size="14" fill="currentColor">
    <text x="0" y="38">Unmodified planner</text>
    <text x="0" y="98">Doors given by the simulator</text>
    <text x="0" y="158">Doors read from the embeddings</text>
    <path d="M250 22 h268.5 a4 4 0 0 1 4 4 v20 a4 4 0 0 1 -4 4 h-268.5 z" fill="var(--muted)"/>
    <path d="M250 82 h320 a4 4 0 0 1 4 4 v20 a4 4 0 0 1 -4 4 h-320 z" fill="var(--accent)"/>
    <path d="M250 142 h320 a4 4 0 0 1 4 4 v20 a4 4 0 0 1 -4 4 h-320 z" fill="var(--accent)"/>
    <text x="590" y="38" font-weight="600">67</text>
    <text x="590" y="98" font-weight="600">80</text>
    <text x="590" y="158" font-weight="600">80</text>
  </g>
  <line x1="250" y1="10" x2="250" y2="185" stroke="var(--line)" stroke-width="1"/>
</svg>
<figcaption>Trips reaching the goal, out of 80 fresh trips. Same trips for every version.</figcaption>
</figure>

| Version | Reached the goal |
|---|---|
| Unmodified planner | 67 / 80 |
| Doors given by the simulator | 80 / 80 |
| Doors read from the embeddings | 80 / 80 |

Paired on the same trips, the embedding-based version beat the unmodified planner on 13 trips and lost on none (exact sign test, p below 0.001). It matched the simulator-supplied version on every trip. The route it read from the embeddings was the correct one on all 80 trips. An earlier confirmation on 80 other fresh trips, with the doors supplied by the simulator, gave the same direction (80 against 68). So the effect is real *on this checkpoint*; the replication section below shows it did not carry over to the other two.

## Two trips, side by side

The same trip, the same start and goal, the same closed door. Left: the unmodified planner. Right: the planner steered through the doors read from the embeddings. The red bar is the closed door, the numbered rings are the two doors of the route, and each line is the agent's recorded path.

<figure>
<video controls muted loop playsinline preload="metadata" poster="{{ '/assets/images/trip_278.png' | relative_url }}">
  <source src="{{ '/assets/video/trip_278.mp4' | relative_url }}" type="video/mp4">
</video>
<figcaption>Trip 278. The unmodified planner spends the budget pushing against the closed door. The steered planner goes through door 1, then door 2, and reaches the goal at action 153.</figcaption>
</figure>

<figure>
<video controls muted loop playsinline preload="metadata" poster="{{ '/assets/images/trip_62.png' | relative_url }}">
  <source src="{{ '/assets/video/trip_62.mp4' | relative_url }}" type="video/mp4">
</video>
<figcaption>Trip 62, a different layout. The unmodified planner takes the shorter route, through the right door, and then runs into the closed bottom door. The steered planner takes the long way round and reaches the goal at action 168.</figcaption>
</figure>

I picked these two from the 13 trips where the unmodified planner missed and the steered one reached the goal, so they show the failure mode, not a typical trip. On the other 67 trips both reached the goal. The closed door in each video is the one the embedding reader reported, and the route it implied matched the simulator's route on all 80 trips.

## Does the doorway itself matter?

An obvious objection: reaching any spot in the next room also forces the agent through the door, so maybe *any* subgoal there would do. I tested that. On the same 80 trips, with the same planner, steering and scoring, I replaced the discovered anchors by random ones: for each door, the same number of anchors, each placed at a random spot in the room behind it (at least 30 px from every door), with the same tolerances. I wrote down the decision rule before the run, and I thought it fairly likely that the random anchors would do about as well.

<figure>
<svg viewBox="0 0 640 200" role="img" aria-label="Trips reaching the goal out of 80: unmodified planner 67, random anchors in the next room 67, discovered door anchors 80." width="100%" style="max-width:640px">
  <g font-family="system-ui, sans-serif" font-size="14" fill="currentColor">
    <text x="0" y="38">Unmodified planner</text>
    <text x="0" y="98">Random anchors in the next room</text>
    <text x="0" y="158">Discovered door anchors</text>
    <path d="M250 22 h268.5 a4 4 0 0 1 4 4 v20 a4 4 0 0 1 -4 4 h-268.5 z" fill="var(--muted)"/>
    <path d="M250 82 h268.5 a4 4 0 0 1 4 4 v20 a4 4 0 0 1 -4 4 h-268.5 z" fill="var(--muted)"/>
    <path d="M250 142 h320 a4 4 0 0 1 4 4 v20 a4 4 0 0 1 -4 4 h-320 z" fill="var(--accent)"/>
    <text x="590" y="38" font-weight="600">67</text>
    <text x="590" y="98" font-weight="600">67</text>
    <text x="590" y="158" font-weight="600">80</text>
  </g>
  <line x1="250" y1="10" x2="250" y2="185" stroke="var(--line)" stroke-width="1"/>
</svg>
<figcaption>Trips reaching the goal, out of 80. Same trips, same planner, same scoring rule. The discovered anchors here use the simulator's route, as in the earlier table.</figcaption>
</figure>

The discovered anchors beat the random ones on 13 trips and lost on none (p below 0.001). The random anchors did no better than the unmodified planner: they changed *which* trips succeeded (12 gained, 12 lost) but not how many. So in this setup the position of the anchors matters, and steering towards "somewhere in the next room" is not enough.

This covers one family of random anchors. The random spots are also farther from the door than the discovered ones, so anchors placed *near* a door but not on it would be a stricter test, and I have not run it.

## Repeating it on two more trained copies

The model I used is one trained copy (seed 42). The authors report three, so I ran the same test on the other two (seeds 43 and 44). These are separately trained copies of the same recipe, in the same world with the same trips. Before any episode of these runs existed I wrote down the rules: discovery has to find the top, left and right doors with a shuffle test p below 0.01; then the steered planner has to solve at least 72 of 80 trips and beat the unmodified planner on a paired sign test; and if the unmodified planner already solves more than 74 of 80, the verdict is "not testable", because no gain could then be significant. (I first wrote 75 and corrected it to 74 after a unit test showed that five one-sided trips give p = 0.0625. That was before any episode ran.)

**Discovery replicated.** On both copies the same pattern appeared: top, left and right doors found on held-out episodes, the bottom door not, shuffle test p = 0.001.

**Planning did not, cleanly.** Each row is 80 trips, with the planner steered through the discovered anchors using the simulator's route, as in the first test.

| Checkpoint | Unmodified | Steered | Trips only one arm solved | Verdict under the fixed rule |
|---|---|---|---|---|
| 42 (first test) | 67 | 80 | steered 13, unmodified 0 | supported |
| 43 | 54 | 43 | steered 10, unmodified 21 (p = 0.07) | not replicated |
| 44 | 75 | 79 | steered 5, unmodified 1 (p = 0.22) | not testable |

The random-anchor control, run on the same trips, also split. On copy 43 the discovered anchors were no better than random ones (outcome C, p = 0.08). On copy 44 they beat random anchors, 10 trips to 1 (p = 0.01), even though the main test could not show a gain.

Two observations I made after seeing the results, so treat them as leads and not as findings. On copy 43 the steering constraint was satisfied in 65% of a trip's planning solves on average (59% of all solves pooled; 34% on the trips the run's own judge counted as failures), against 100% and 99.7% on the other two. I do not know why; the tolerance around each anchor was chosen on copy 42 and not re-tuned. Applying the same unchanged selection rule to copy 43's own discovery data gives a looser tolerance than copy 42's (0.75 instead of 0.5), because the tighter one covers too few door visits on that copy. I tested that with a small exploratory run at the looser value, with the rule written first. It moved the steered planner from clearly behind the unmodified one (43 against 54 of 80) to level with it (53 against 54), and the constraint was satisfied more often (76% per trip instead of 65%), but it did not give a gain. So the tolerance looks like part of the problem on this copy, not the whole of it. It was run on trips I had already seen, so it is a lead and not a replication. And on copy 44 a stricter crossing judge credited the unmodified planner with 30 trips and the steered one with 79, which I cannot explain yet. I also have not independently verified the baseline copies I made for the two new runs.

## Attempts without door labels, and why they failed

The clustering uses no labels, but choosing *which* cluster is a door still uses the simulator's record of which door each trip crossed. I tried to remove that. The plan, written down before each run: build four regions from the embeddings alone, group each trip by the regions of its first and last state, and run the same door search with those groups in place of the door labels. Each time I first checked that the labelled version, run through the same code, reproduced the earlier result.

**Attempt 1: four k-means regions.** The regions did not line up with the four rooms. Only about half of the states sat in the majority room of their region.

**Attempt 2: regions from which states follow which.** The idea is that rooms are dense parts of the graph of "this state follows that one" within successful trips, and doors are the narrow places that join them. This did worse: not a single transition crossed from one region to another, and every trip stayed inside one region from its first state to its last. The regions sorted the trips by *which door is closed*, while the rooms of the trips were mixed in every region.

**Attempt 3: remove the layout first.** Those islands are a label-free way to find the layouts, and they matched the closed door perfectly. So I subtracted each layout's average embedding and repeated attempt 2. The labelled analysis still found its three doors on the corrected embeddings, but the regions still failed. The layouts do not differ by a constant shift: the trips with no closed door stayed a separate island from the rest. And only about a quarter of the embedding variance lies *within* a trip, which is where position changes.

So in this model the upper embedding is dominated by what differs between trips, and the agent's position is a smaller part of it. That is a fact about one frozen model on this data, not a general claim about H-JEPA. Recovering the layout without labels worked; recovering the rooms and doors did not. Each attempt used the same data and was designed after seeing why the one before it failed, so a replication on new episodes would still be needed.

**Attempts 4 and 5: a fixed house, then a replication.** Since layout dominates the embedding, I removed it as a variable: the same house (bottom door closed) and the same goal, with only the start changing, 80 starts from each of three rooms. Idea: if doorway states are real, many different trips pass through the same few groups of similar states, and I can look for places where a trip enters such a group, without any door labels. I fixed the rules first: 64 k-means groups of upper embeddings, coverage counted in distinct trips, an "entry" where smoothed coverage rises sharply, and success if entries sit within 20 px of an open door more often than random states from the same trips (outcome A needs at least 60% at doors, p below 0.01, and at least two doors). A toy world with a perfect position embedding showed the detector works while noise is below about a quarter of the signal, so a negative result is only weakly informative.

The first run gave outcome A. On fresh trips with all three trained copies, the replication rule (outcome A on the main split and on at least 6 of 8 random splits) failed on every copy: 2 of 8 splits on copy 42 and 3 of 8 on copies 43 and 44. So the positive result does not stand as a finding.

Looking at the data afterwards, which does not change that verdict: on every copy, 7 of 8 splits had entries at doors more often than random points on the same trips (typically 60 to 90% against about 43%). The detector failed by finding entries on only about half the trips, and it mostly finds the first door of a route, the left and right doors much more than the top. That is a weak, consistent hint that door-like merge points exist in the embedding, and no more. Any new detector would need its own rule written first and fresh trips.

## The catch

An earlier, stricter rule asks the agent to go a fixed distance past each doorway before it counts as crossed. On that rule the embedding-based version passes 72 of 80 trips, against 80 of 80 for the version given the doors. On the other 8 trips it still reached the goal through the right doors, ending within a few pixels of it, but it does not cross the doorway as cleanly. My untested guess is that the crossing detector releases a door a few actions too early.

I changed the scoring rule after seeing earlier results. That is why the last run used new trips and a rule fixed beforehand, and why I report both rules.

## What this does not show

- **One world, one recipe.** Three trained copies of one model in one four-room environment. The planning benefit appeared on one of them, and the cause of the difference is unexplained.
- **Labels in discovery.** The clustering uses no labels, but choosing *which* cluster is a door still uses the simulator's record of which door each episode crossed. My attempts to remove that failed or did not replicate (see above), so this is not label-free discovery. The route, the order of the doors and the crossing check in the planning tests also still come from the simulator.
- **Narrow trips.** Trips through three of the four doors, each with one door closed.
- **One planner seed per trip.**
- **One family of random anchors.** The control rules out "any spot in the next room", not every other kind of waypoint (evenly spaced ones, or random spots near the door).

## What is next

1. Understand why steering helped on one trained copy and not on another. The tolerance explains part of it on copy 43 but not a gain; about a third of the planner's solves there still fail the constraint, for reasons I have not found. A clean test would use fresh trips, with each copy's tolerance set by the same rule beforehand.
2. A stricter control: random anchors placed near a door but not on it.
3. Finding the doors without labels is still open. A new detector would need a new rule and fresh trips, and an idea beyond tuning the last one.
4. A second environment.

The code is not published yet. I will add it, with the scoring rules and predeclared tests, when it is ready.
