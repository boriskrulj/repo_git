---
title: "Where successful plans squeeze through: door checkpoints in a frozen H-JEPA"
date: 2026-11-21
tags: [world-models, planning, jepa, lab-notebook]
excerpt: "A lab notebook on finding recurring states in a frozen hierarchical world model and steering its planner through them. What worked, what didn't, and what is still open."
---

*Status: draft lab notebook. One frozen model, one small world, results still being checked. Nothing here is a finished claim.*

## The question

H-JEPA (Zhang, Terver, Rabbat, LeCun, Balestriero; arXiv:2610.06805) trains a hierarchy of action-conditioned world models. The upper level predicts further ahead in its own latent space, and planning runs top-down: the upper level proposes a subgoal and the lower level plans towards it. The authors release their code at [kevinghst/H-JEPA](https://github.com/kevinghst/H-JEPA).

I took one of those frozen two-level models, trained for the four-room navigation task, and asked a narrow question:

> If I look at many *successful* episodes and ask which states recur, do the recurring states act as useful checkpoints for the planner?

I never retrain the model. The model still predicts outcomes and the planner still replans after every few steps. The only thing I change is a constraint on which plans the planner prefers.

## The idea

In a four-room world every trip between rooms has to squeeze through a doorway. Inside a room, successful episodes take many different paths. At the doorway they all look alike. A state that every successful episode on the same route shares, and that few other states share, is a natural checkpoint.

## What I did

1. Cluster the upper-level embeddings of states from successful episodes (k-means, 16 clusters).
2. Pick the clusters that many distinct episodes of the same route visit, counting episodes and not frames, and leaving out the last few states before the goal. A comparison against neighbouring doors keeps clusters that merely sit inside a room from being mistaken for doors.
3. Hold out episodes to check that those clusters still recur on trips the clustering never saw.
4. Steer the planner: rank candidate plans by how well they pass through the checkpoints in order, then by distance to the goal, and always execute the best plan.

## What came out

On wall-blocked trips, where one door is closed and the straight route is blocked, the planner steered through the discovered door clusters reached the goal more often than the unmodified planner.

I also removed some of the hand-fed information step by step. The route and the moment a door was crossed were read from the model's own embeddings, with readers taught from earlier episodes.

On 80 fresh trips, scored by a rule I wrote down before the run (within 16 px of the goal, with the true doors passed in order):

| Version | Reached the goal |
|---|---|
| Unmodified planner | 67 / 80 |
| Doors given by the simulator | 80 / 80 |
| Doors read from the embeddings | 80 / 80 |

The embedding-based version beat the unmodified planner on 13 trips and lost on none. The route read from the embeddings was correct on all 80 trips.

## The catch

An earlier, stricter rule asked the agent to go a fixed distance past each doorway. On that rule the embedding-based version passed 72 of 80 trips against 80 of 80 for the version given the doors. On those 8 trips it still reached the goal through the right doors, so it gets there as reliably but does not cross doorways as cleanly. I changed the scoring rule after seeing earlier results, which is why the last run used new trips and a rule fixed beforehand.

## What this does not show

- **One model, one world.** A single frozen checkpoint and one four-room environment.
- **Labels in discovery.** Choosing which cluster is a door still uses the simulator's note of which door each episode crossed. The clustering does not.
- **Narrow trips.** Trips through three of the four doors, each with one closed door.
- **One planner seed per trip.**
- **Doorway or just room?** Reaching any spot in the next room also forces the door. I am running a control with random anchors in the next room to see whether the doorway position itself matters.

## What is next

1. The random-anchor control (running).
2. Finding the bottlenecks without any door labels, for example as narrow points where the map of clusters would split.
3. More checkpoints (seeds) and a second environment.

I will update this post as results come in.
