---
layout: post
title: "Learning checkpoints from experience, and planning with them alone"
date: 2026-11-24
tags: [world-models, planning, jepa, lab-notebook]
description: "A frozen world-model planner learns a few repeated states from its own successful trips, throws the trips away, and plans new trips with those states alone. Failures drop from about one in five to about one in eighty."
excerpt: "A frozen world-model planner learns a few repeated states from its own successful trips, throws the trips away, and plans new trips with those states alone."
---

*Status: ongoing project, one small world, two trained models. This post replaces the [earlier lab notebook]({% post_url 2026-11-21-door-checkpoints-in-a-frozen-h-jepa %}) as the summary of where things stand. Last update: 24 November 2026.*

## The objective

There are many ways to reach a goal, but successful attempts keep passing through the same few states. Think of driving from Brussels to Paris: every route is different in the streets, but nearly all of them cross the same border and the same ring road. Those shared states are natural **checkpoints**.

The idea I wanted to test:

1. A robot can **learn** those repeated states by looking at its own successful experience, without anyone labelling them.
2. It can then **plan new trips with the checkpoints alone**, without remembering the trips it learned them from.
3. And they help **because** they are repeated, not just because any hint helps.

The long-term goal is a hierarchy: checkpoints of checkpoints, with as many levels as a task needs. This post is about the first level.

## The setup

I use a frozen, already-trained world model, H-JEPA (Zhang, Terver, Rabbat, LeCun, Balestriero; [arXiv:2610.06805](https://arxiv.org/abs/2610.06805), code at [kevinghst/H-JEPA](https://github.com/kevinghst/H-JEPA)), trained for a four-room navigation task. The robot sees only pictures. One of the four doors may be closed, so some trips have to go the long way round, through two doors. The unmodified planner often heads the wrong way and gets stuck at the closed door.

I never retrain the model. The planner still predicts and replans as before. The only change is that it receives two hints, "pass through here, then here", and prefers plans that follow them.

## How it works

**Learn.** The robot's experience is 1,730 short successful trips, each through a single door. For each trip, I find the state where it comes closest to other similar trips: the most shared point. Then I group those states by how similar the model finds them, with no labels. The groups formed by themselves: exactly **16**, on both models. They turned out to be every open door under every layout (four with no door closed, three for each of the four closed-door layouts). Each learned checkpoint also keeps a summary of the two regions it connects, like a door joins two rooms. After that, the 1,730 trips are thrown away.

<figure>
<svg viewBox="0 0 616 156" role="img" aria-label="The 16 learned checkpoints, shown on the map for each of the five layouts: every open door in every layout, and nothing else." width="100%" style="max-width:801px">
<g font-family="system-ui, sans-serif" font-size="9" fill="currentColor">
<rect x="0" y="6" width="112" height="112" rx="4" fill="none" stroke="var(--line)"/>
<line x1="56.0" y1="6.0" x2="56.0" y2="26.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="56.0" y1="42.0" x2="56.0" y2="82.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="56.0" y1="98.0" x2="56.0" y2="118.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="0.0" y1="62.0" x2="20.0" y2="62.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="36.0" y1="62.0" x2="76.0" y2="62.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="92.0" y1="62.0" x2="112.0" y2="62.0" stroke="currentColor" stroke-width="2.2"/>
<circle cx="55.6" cy="35.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.2" cy="34.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="53.1" cy="34.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.0" cy="33.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="53.8" cy="34.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.5" cy="33.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="69.7" cy="36.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.2" cy="35.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="63.0" cy="32.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.9" cy="36.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="48.1" cy="29.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.4" cy="33.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.5" cy="32.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.7" cy="32.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="54.6" cy="36.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="54.3" cy="34.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="61.1" cy="32.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="46.6" cy="34.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="54.9" cy="35.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="58.1" cy="31.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="46.2" cy="37.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.0" cy="31.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.3" cy="34.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.1" cy="34.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.2" cy="32.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.0" cy="32.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="53.4" cy="37.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="62.6" cy="35.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="54.4" cy="29.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.8" cy="36.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.4" cy="31.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="51.1" cy="37.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="58.4" cy="40.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="54.9" cy="32.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="46.7" cy="38.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.8" cy="32.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="60.2" cy="32.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.8" cy="33.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.9" cy="33.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.4" cy="35.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.7" cy="35.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.2" cy="31.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.6" cy="37.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.3" cy="34.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.6" cy="31.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.4" cy="34.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="54.0" cy="31.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="53.9" cy="31.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.5" cy="36.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="65.6" cy="33.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.8" cy="36.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.1" cy="33.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.9" cy="31.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.3" cy="36.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.8" cy="35.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.3" cy="32.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.5" cy="32.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.5" cy="34.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="54.7" cy="37.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="59.0" cy="34.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.3" cy="33.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="44.6" cy="39.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.4" cy="32.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.9" cy="33.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.5" cy="32.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.0" cy="35.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<path d="M55.8 29.8 L60.3 34.3 L55.8 38.8 L51.3 34.3 Z" fill="var(--bg, #fff)" stroke="var(--accent)" stroke-width="1.6"/>
<circle cx="82.2" cy="62.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="85.5" cy="62.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="83.5" cy="61.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="86.7" cy="61.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="85.5" cy="61.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="85.9" cy="63.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="84.3" cy="59.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="85.2" cy="62.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="79.5" cy="63.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="81.3" cy="61.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="83.9" cy="61.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="80.2" cy="64.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="85.9" cy="64.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="80.5" cy="58.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="80.8" cy="60.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="82.0" cy="62.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="83.4" cy="61.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="82.5" cy="60.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="84.3" cy="58.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="83.6" cy="61.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="83.5" cy="62.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="85.7" cy="61.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="82.2" cy="60.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="85.9" cy="61.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="81.5" cy="62.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="86.4" cy="61.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="84.6" cy="61.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="84.2" cy="62.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="81.0" cy="68.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="82.6" cy="63.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="79.2" cy="65.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="84.2" cy="62.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="81.0" cy="60.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="84.8" cy="62.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="85.4" cy="61.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="82.3" cy="64.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="86.4" cy="64.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="85.8" cy="62.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="80.7" cy="71.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="84.4" cy="61.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="81.5" cy="68.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="83.3" cy="61.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="83.3" cy="61.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="82.8" cy="67.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="86.1" cy="61.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="85.3" cy="62.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="81.8" cy="62.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="83.5" cy="58.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="81.9" cy="62.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="81.1" cy="64.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="82.6" cy="62.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="81.2" cy="63.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="86.2" cy="60.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="82.4" cy="63.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="84.1" cy="62.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="80.1" cy="60.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="81.5" cy="62.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="84.8" cy="62.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="81.4" cy="60.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="81.4" cy="61.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="86.0" cy="62.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="83.3" cy="62.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="84.1" cy="62.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="82.1" cy="61.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="81.6" cy="73.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="84.9" cy="62.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="78.2" cy="63.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="87.5" cy="61.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="81.0" cy="60.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="85.8" cy="62.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="85.9" cy="64.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="84.6" cy="62.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="84.6" cy="62.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<path d="M83.5 58.2 L88.0 62.7 L83.5 67.2 L79.0 62.7 Z" fill="var(--bg, #fff)" stroke="var(--accent)" stroke-width="1.6"/>
<circle cx="57.4" cy="84.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.5" cy="86.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.4" cy="91.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="70.6" cy="87.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.0" cy="91.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="58.2" cy="89.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.4" cy="88.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.0" cy="91.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.6" cy="91.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.0" cy="89.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.3" cy="87.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="51.4" cy="87.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.1" cy="92.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="58.1" cy="85.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.0" cy="90.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="46.8" cy="95.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.5" cy="91.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.6" cy="91.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="58.0" cy="87.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.4" cy="91.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.0" cy="90.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.4" cy="87.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="49.9" cy="82.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="53.9" cy="86.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="53.0" cy="92.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.6" cy="85.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.8" cy="90.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.0" cy="94.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="59.7" cy="88.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.7" cy="94.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.6" cy="88.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.2" cy="92.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.4" cy="91.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="59.6" cy="87.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.1" cy="91.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="54.8" cy="87.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="50.4" cy="87.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.8" cy="90.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.9" cy="90.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="60.0" cy="91.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.6" cy="87.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.6" cy="90.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="54.9" cy="91.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.2" cy="90.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.9" cy="92.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.2" cy="89.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="67.2" cy="85.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.2" cy="88.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.7" cy="88.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="63.0" cy="91.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="54.8" cy="90.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.0" cy="88.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.1" cy="92.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.6" cy="89.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.2" cy="88.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.9" cy="91.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="55.6" cy="91.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.5" cy="87.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="44.5" cy="86.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.5" cy="92.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.2" cy="87.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="57.7" cy="91.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="51.9" cy="95.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="56.6" cy="90.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<path d="M55.9 84.8 L60.4 89.3 L55.9 93.8 L51.4 89.3 Z" fill="var(--bg, #fff)" stroke="var(--accent)" stroke-width="1.6"/>
<circle cx="29.4" cy="62.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="30.3" cy="77.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.5" cy="64.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="31.1" cy="67.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="30.1" cy="63.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="26.3" cy="61.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.8" cy="59.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="26.5" cy="62.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="30.2" cy="61.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="29.5" cy="62.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="31.5" cy="62.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="26.1" cy="61.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="31.9" cy="64.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="30.3" cy="63.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="28.6" cy="61.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="25.5" cy="59.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="30.2" cy="62.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="25.9" cy="61.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.1" cy="61.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="29.8" cy="61.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="26.0" cy="66.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="26.5" cy="62.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.7" cy="66.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.2" cy="62.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="30.6" cy="63.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="35.6" cy="45.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.5" cy="61.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="26.2" cy="62.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.8" cy="61.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.8" cy="62.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="26.7" cy="50.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="30.5" cy="60.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="26.2" cy="62.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="28.6" cy="72.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="30.4" cy="61.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.0" cy="62.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.5" cy="62.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="30.6" cy="62.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.2" cy="61.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.0" cy="62.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="25.9" cy="62.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="22.6" cy="60.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="29.3" cy="62.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="29.2" cy="72.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="26.0" cy="51.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="26.9" cy="62.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="30.3" cy="62.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="23.5" cy="63.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="29.7" cy="61.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="32.7" cy="52.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="25.0" cy="51.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="26.2" cy="62.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="30.0" cy="62.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="31.0" cy="75.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="28.0" cy="62.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.2" cy="61.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="28.4" cy="63.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="30.2" cy="61.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="26.5" cy="62.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="29.6" cy="63.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="30.8" cy="62.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="29.9" cy="62.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="29.9" cy="62.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="29.6" cy="63.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="30.6" cy="63.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="28.7" cy="62.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.3" cy="69.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="26.9" cy="63.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.2" cy="62.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="29.9" cy="60.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.5" cy="61.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="28.5" cy="62.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="25.8" cy="51.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="30.1" cy="61.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.6" cy="61.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="26.0" cy="62.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="26.3" cy="62.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="28.0" cy="61.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="26.0" cy="60.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="27.2" cy="61.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<path d="M28.4 57.7 L32.9 62.2 L28.4 66.7 L23.9 62.2 Z" fill="var(--bg, #fff)" stroke="var(--accent)" stroke-width="1.6"/>
<text x="56.0" y="134" text-anchor="middle">no door closed</text>
<rect x="126" y="6" width="112" height="112" rx="4" fill="none" stroke="var(--line)"/>
<line x1="182.0" y1="6.0" x2="182.0" y2="26.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="182.0" y1="42.0" x2="182.0" y2="82.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="182.0" y1="98.0" x2="182.0" y2="118.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="126.0" y1="62.0" x2="146.0" y2="62.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="162.0" y1="62.0" x2="202.0" y2="62.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="218.0" y1="62.0" x2="238.0" y2="62.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="182.0" y1="26.0" x2="182.0" y2="42.0" stroke="#b4412e" stroke-width="3"/>
<circle cx="156.2" cy="58.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="155.4" cy="64.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="152.5" cy="74.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="152.8" cy="64.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="154.0" cy="62.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="153.7" cy="61.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="157.3" cy="64.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="151.9" cy="64.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="156.5" cy="64.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="153.2" cy="61.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="149.2" cy="77.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="153.7" cy="63.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="154.4" cy="64.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="153.0" cy="62.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="156.5" cy="61.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="154.1" cy="63.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="155.9" cy="62.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="152.7" cy="64.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="153.0" cy="60.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="154.1" cy="63.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="155.5" cy="64.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="153.3" cy="62.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="153.3" cy="63.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="154.0" cy="63.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="155.6" cy="63.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<path d="M154.2 58.0 L158.7 62.5 L154.2 67.0 L149.7 62.5 Z" fill="var(--bg, #fff)" stroke="var(--accent)" stroke-width="1.6"/>
<circle cx="182.7" cy="88.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="180.1" cy="87.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="183.0" cy="86.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="180.4" cy="90.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="181.7" cy="91.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="180.9" cy="90.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="180.2" cy="90.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="182.5" cy="92.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="181.7" cy="86.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="181.5" cy="87.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="182.6" cy="92.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="181.8" cy="88.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="179.0" cy="88.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="181.5" cy="89.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="173.0" cy="94.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="181.0" cy="92.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="187.2" cy="91.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="182.6" cy="91.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="180.5" cy="90.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="180.9" cy="90.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="180.9" cy="90.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="180.9" cy="91.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="181.4" cy="89.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="183.3" cy="91.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="182.4" cy="88.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<path d="M181.4 85.9 L185.9 90.4 L181.4 94.9 L176.9 90.4 Z" fill="var(--bg, #fff)" stroke="var(--accent)" stroke-width="1.6"/>
<circle cx="208.0" cy="61.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="209.5" cy="63.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="208.3" cy="61.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="210.4" cy="62.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="211.7" cy="63.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="208.7" cy="62.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="208.8" cy="62.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="211.6" cy="63.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="209.3" cy="62.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="207.7" cy="63.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="212.4" cy="65.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="209.3" cy="63.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="212.6" cy="60.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="210.2" cy="62.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="208.5" cy="62.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="212.0" cy="63.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="207.7" cy="62.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="208.6" cy="63.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="212.3" cy="66.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="212.8" cy="61.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="208.8" cy="62.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="208.5" cy="59.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="210.8" cy="59.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="210.3" cy="63.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<path d="M209.4 58.3 L213.9 62.8 L209.4 67.3 L204.9 62.8 Z" fill="var(--bg, #fff)" stroke="var(--accent)" stroke-width="1.6"/>
<text x="182.0" y="134" text-anchor="middle">top door closed</text>
<rect x="252" y="6" width="112" height="112" rx="4" fill="none" stroke="var(--line)"/>
<line x1="308.0" y1="6.0" x2="308.0" y2="26.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="308.0" y1="42.0" x2="308.0" y2="82.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="308.0" y1="98.0" x2="308.0" y2="118.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="252.0" y1="62.0" x2="272.0" y2="62.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="288.0" y1="62.0" x2="328.0" y2="62.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="344.0" y1="62.0" x2="364.0" y2="62.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="308.0" y1="82.0" x2="308.0" y2="98.0" stroke="#b4412e" stroke-width="3"/>
<circle cx="308.4" cy="33.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="307.0" cy="34.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="309.0" cy="34.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="309.6" cy="33.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="308.7" cy="36.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="307.4" cy="33.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="308.4" cy="32.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="305.8" cy="36.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="306.2" cy="31.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="310.5" cy="32.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="306.1" cy="32.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="305.9" cy="30.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="307.0" cy="32.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="309.8" cy="38.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="310.9" cy="34.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="306.8" cy="32.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="308.1" cy="33.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="307.2" cy="32.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="307.9" cy="35.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="323.8" cy="39.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="307.5" cy="33.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="306.0" cy="31.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<path d="M308.3 29.5 L312.8 34.0 L308.3 38.5 L303.8 34.0 Z" fill="var(--bg, #fff)" stroke="var(--accent)" stroke-width="1.6"/>
<circle cx="335.9" cy="61.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="333.6" cy="59.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="337.1" cy="62.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="333.1" cy="63.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="336.9" cy="64.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="335.7" cy="61.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="335.7" cy="59.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="334.4" cy="60.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="338.3" cy="62.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="336.7" cy="62.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="337.0" cy="62.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="333.1" cy="61.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="337.1" cy="60.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="336.4" cy="62.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="335.6" cy="61.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="337.1" cy="62.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="336.1" cy="66.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="336.4" cy="61.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="336.9" cy="62.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="337.6" cy="61.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="338.5" cy="62.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="337.3" cy="62.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="333.6" cy="64.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="336.9" cy="63.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="335.6" cy="62.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<path d="M336.1 57.8 L340.6 62.3 L336.1 66.8 L331.6 62.3 Z" fill="var(--bg, #fff)" stroke="var(--accent)" stroke-width="1.6"/>
<circle cx="280.1" cy="63.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="279.4" cy="61.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="280.7" cy="63.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="281.5" cy="63.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="281.4" cy="62.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="279.5" cy="61.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="279.3" cy="61.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="279.8" cy="62.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="279.2" cy="61.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="285.2" cy="61.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="281.3" cy="65.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="279.1" cy="62.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="279.3" cy="63.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="280.3" cy="62.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="281.3" cy="63.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="280.6" cy="63.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="280.3" cy="62.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="278.3" cy="61.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="280.3" cy="63.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="279.7" cy="62.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="279.4" cy="62.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="277.9" cy="62.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<path d="M280.3 58.1 L284.8 62.6 L280.3 67.1 L275.8 62.6 Z" fill="var(--bg, #fff)" stroke="var(--accent)" stroke-width="1.6"/>
<text x="308.0" y="134" text-anchor="middle">bottom door closed</text>
<rect x="378" y="6" width="112" height="112" rx="4" fill="none" stroke="var(--line)"/>
<line x1="434.0" y1="6.0" x2="434.0" y2="26.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="434.0" y1="42.0" x2="434.0" y2="82.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="434.0" y1="98.0" x2="434.0" y2="118.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="378.0" y1="62.0" x2="398.0" y2="62.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="414.0" y1="62.0" x2="454.0" y2="62.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="470.0" y1="62.0" x2="490.0" y2="62.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="398.0" y1="62.0" x2="414.0" y2="62.0" stroke="#b4412e" stroke-width="3"/>
<circle cx="432.0" cy="32.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="433.1" cy="32.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="432.2" cy="33.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="431.8" cy="33.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="432.8" cy="33.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="437.9" cy="36.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="435.7" cy="35.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="433.4" cy="36.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="432.1" cy="33.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="434.9" cy="32.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="432.1" cy="33.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="430.1" cy="32.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="433.6" cy="36.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="431.7" cy="36.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="431.8" cy="31.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="431.8" cy="34.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="434.5" cy="35.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="432.0" cy="33.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="433.9" cy="39.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="433.0" cy="36.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="436.2" cy="32.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="438.2" cy="30.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="431.7" cy="34.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="426.5" cy="37.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<path d="M432.1 29.4 L436.6 33.9 L432.1 38.4 L427.6 33.9 Z" fill="var(--bg, #fff)" stroke="var(--accent)" stroke-width="1.6"/>
<circle cx="424.6" cy="90.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="433.5" cy="90.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="434.3" cy="91.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="431.6" cy="89.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="433.1" cy="89.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="434.1" cy="90.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="433.6" cy="89.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="433.6" cy="89.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="433.2" cy="88.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="434.8" cy="88.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="435.4" cy="85.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="434.8" cy="89.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="435.3" cy="92.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="433.4" cy="89.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="434.3" cy="88.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="434.5" cy="90.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="430.3" cy="88.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="430.1" cy="84.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="433.9" cy="90.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="431.5" cy="91.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="435.4" cy="90.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="437.8" cy="87.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="436.3" cy="91.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="432.6" cy="88.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="433.2" cy="89.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="434.8" cy="90.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="432.9" cy="91.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<path d="M433.5 85.7 L438.0 90.2 L433.5 94.7 L429.0 90.2 Z" fill="var(--bg, #fff)" stroke="var(--accent)" stroke-width="1.6"/>
<circle cx="458.7" cy="63.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="462.8" cy="59.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="462.7" cy="61.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="460.7" cy="63.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="463.2" cy="61.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="465.2" cy="58.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="460.3" cy="64.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="460.6" cy="60.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="461.0" cy="60.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="463.9" cy="58.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="459.3" cy="73.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="460.0" cy="60.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="460.7" cy="60.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="463.3" cy="63.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="459.9" cy="55.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="460.2" cy="61.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="462.8" cy="64.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="460.6" cy="59.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="461.6" cy="61.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="462.1" cy="62.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="463.5" cy="64.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="463.2" cy="62.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="463.2" cy="61.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="461.2" cy="60.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="460.6" cy="63.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<path d="M461.3 57.5 L465.8 62.0 L461.3 66.5 L456.8 62.0 Z" fill="var(--bg, #fff)" stroke="var(--accent)" stroke-width="1.6"/>
<text x="434.0" y="134" text-anchor="middle">left door closed</text>
<rect x="504" y="6" width="112" height="112" rx="4" fill="none" stroke="var(--line)"/>
<line x1="560.0" y1="6.0" x2="560.0" y2="26.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="560.0" y1="42.0" x2="560.0" y2="82.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="560.0" y1="98.0" x2="560.0" y2="118.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="504.0" y1="62.0" x2="524.0" y2="62.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="540.0" y1="62.0" x2="580.0" y2="62.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="596.0" y1="62.0" x2="616.0" y2="62.0" stroke="currentColor" stroke-width="2.2"/>
<line x1="580.0" y1="62.0" x2="596.0" y2="62.0" stroke="#b4412e" stroke-width="3"/>
<circle cx="558.2" cy="35.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="557.8" cy="34.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="562.7" cy="32.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="562.2" cy="35.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="559.8" cy="34.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="558.4" cy="35.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="560.9" cy="33.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="559.5" cy="33.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="557.7" cy="35.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="559.4" cy="35.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="561.2" cy="33.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="561.3" cy="33.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="559.8" cy="33.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="557.7" cy="34.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="561.5" cy="33.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="561.5" cy="32.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="560.4" cy="33.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="561.8" cy="32.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="558.2" cy="34.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="563.1" cy="35.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="559.8" cy="34.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="558.2" cy="32.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="558.3" cy="35.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="557.2" cy="34.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="561.2" cy="33.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="561.7" cy="34.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="561.8" cy="33.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<path d="M559.8 30.3 L564.3 34.8 L559.8 39.3 L555.3 34.8 Z" fill="var(--bg, #fff)" stroke="var(--accent)" stroke-width="1.6"/>
<circle cx="557.3" cy="93.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="555.3" cy="91.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="559.2" cy="89.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="560.9" cy="88.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="559.5" cy="87.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="561.9" cy="92.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="559.6" cy="88.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="557.9" cy="92.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="560.5" cy="88.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="560.2" cy="88.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="559.6" cy="91.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="558.0" cy="94.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="560.0" cy="92.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="557.1" cy="91.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="561.6" cy="91.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="561.9" cy="88.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="560.8" cy="92.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="553.9" cy="92.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="558.2" cy="88.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="560.9" cy="90.4" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="560.1" cy="90.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="559.3" cy="89.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="559.2" cy="89.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="561.2" cy="92.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="563.6" cy="90.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="565.6" cy="87.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="558.9" cy="91.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="561.7" cy="92.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<path d="M559.9 85.2 L564.4 89.7 L559.9 94.2 L555.4 89.7 Z" fill="var(--bg, #fff)" stroke="var(--accent)" stroke-width="1.6"/>
<circle cx="535.4" cy="60.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="530.7" cy="61.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="533.9" cy="63.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="531.8" cy="62.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="533.0" cy="61.8" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="533.7" cy="62.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="534.4" cy="60.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="529.7" cy="64.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="532.0" cy="61.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="533.1" cy="61.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="533.4" cy="58.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="535.2" cy="63.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="533.3" cy="64.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="530.5" cy="62.9" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="531.9" cy="63.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="529.8" cy="59.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="532.9" cy="62.0" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="532.0" cy="65.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="532.7" cy="60.1" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="531.7" cy="62.5" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="534.3" cy="62.2" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="533.3" cy="61.7" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="531.8" cy="62.3" r="0.9" fill="var(--accent)" opacity="0.35"/>
<circle cx="533.8" cy="63.6" r="0.9" fill="var(--accent)" opacity="0.35"/>
<path d="M532.5 58.1 L537.0 62.6 L532.5 67.1 L528.0 62.6 Z" fill="var(--bg, #fff)" stroke="var(--accent)" stroke-width="1.6"/>
<text x="560.0" y="134" text-anchor="middle">right door closed</text>
</g></svg>
<figcaption>The 16 learned checkpoints (diamonds) of one model, drawn on the map of the layout they belong to; the small dots are the trip states each group was learned from. The red bar is the closed door. Positions are used only to draw this picture; the robot never sees them.</figcaption>
</figure>

**Use.** For a new trip, the robot looks at the start picture and the goal picture and picks two of the 16 checkpoints: one whose region contains the start, one whose region contains the goal, linked through a shared region in between. Those two become the hints.

## What came out

Every test was written down before it ran, used trips the method had never seen, and was repeated on two separately trained copies of the model. Success means reaching the goal through the correct doors, within 210 actions.

<figure class="chart">
<svg viewBox="0 0 640 206" role="img" aria-label="Model 42: trips reaching the goal out of 80. No hints 66, non-repeated hints 71, learned repeated states 79, chaining remembered trips (all 1,730 kept) 78." width="100%" style="max-width:640px">
<g font-family="system-ui, sans-serif" font-size="14" fill="currentColor">
<text x="0" y="34">No hints</text>
<path d="M250 18 h260.0 a4 4 0 0 1 4 4 v20 a4 4 0 0 1 -4 4 h-260.0 z" fill="var(--muted)"/>
<text x="526.0" y="34" font-weight="600">66</text>
<text x="0" y="78">Non-repeated hints</text>
<path d="M250 62 h280.0 a4 4 0 0 1 4 4 v20 a4 4 0 0 1 -4 4 h-280.0 z" fill="#8a5a9e"/>
<text x="546.0" y="78" font-weight="600">71</text>
<text x="0" y="122">Learned repeated states</text>
<path d="M250 106 h312.0 a4 4 0 0 1 4 4 v20 a4 4 0 0 1 -4 4 h-312.0 z" fill="var(--accent)"/>
<text x="578.0" y="122" font-weight="600">79</text>
<text x="0" y="166">Remembering all trips</text>
<path d="M250 150 h308.0 a4 4 0 0 1 4 4 v20 a4 4 0 0 1 -4 4 h-308.0 z" fill="#9aa3ad"/>
<text x="574.0" y="166" font-weight="600">78</text>
<line x1="250" y1="8" x2="250" y2="196" stroke="var(--line)" stroke-width="1"/>
</g></svg>
<figcaption>Model 42: trips reaching the goal, out of the same 80 fresh trips.</figcaption>
</figure>

<figure class="chart">
<svg viewBox="0 0 640 206" role="img" aria-label="Model 43: trips reaching the goal out of 80. No hints 61, non-repeated hints 67, learned repeated states 79, chaining remembered trips (all 1,730 kept) 75." width="100%" style="max-width:640px">
<g font-family="system-ui, sans-serif" font-size="14" fill="currentColor">
<text x="0" y="34">No hints</text>
<path d="M250 18 h240.0 a4 4 0 0 1 4 4 v20 a4 4 0 0 1 -4 4 h-240.0 z" fill="var(--muted)"/>
<text x="506.0" y="34" font-weight="600">61</text>
<text x="0" y="78">Non-repeated hints</text>
<path d="M250 62 h264.0 a4 4 0 0 1 4 4 v20 a4 4 0 0 1 -4 4 h-264.0 z" fill="#8a5a9e"/>
<text x="530.0" y="78" font-weight="600">67</text>
<text x="0" y="122">Learned repeated states</text>
<path d="M250 106 h312.0 a4 4 0 0 1 4 4 v20 a4 4 0 0 1 -4 4 h-312.0 z" fill="var(--accent)"/>
<text x="578.0" y="122" font-weight="600">79</text>
<text x="0" y="166">Remembering all trips</text>
<path d="M250 150 h296.0 a4 4 0 0 1 4 4 v20 a4 4 0 0 1 -4 4 h-296.0 z" fill="#9aa3ad"/>
<text x="562.0" y="166" font-weight="600">75</text>
<line x1="250" y1="8" x2="250" y2="196" stroke="var(--line)" stroke-width="1"/>
</g></svg>
<figcaption>Model 43: trips reaching the goal, out of the same 80 fresh trips.</figcaption>
</figure>

| | No hints | Non-repeated hints | **Learned repeated states** | Remembering all trips |
|---|---|---|---|---|
| Model 42 | 66 / 80 | 71 / 80 | **79 / 80** | 78 / 80 |
| Model 43 | 61 / 80 | 67 / 80 | **79 / 80** | 75 / 80 |

**1. The learned checkpoints make the planner much more reliable.** Failures dropped from 14 to 1 on one model and from 19 to 1 on the other. Paired on the same trips, the checkpoints won 14 to 1 and 19 to 1 against no hints (exact sign test, p = 0.001 and p &lt; 0.0001).

**2. Sixteen learned states are as good as remembering everything.** An earlier version of the method kept all 1,730 trips and stitched two of them together for each new trip. The 16 checkpoints did at least as well (79 against 78 and 75).

**3. They help because they are repeated.** The control is identical in every way, with the same entries, the same choice and the same tolerance, except that each hint is the *least* shared point of a remembered trip instead of the most shared one. Non-repeated hints barely helped (71 and 67), and the repeated states beat them on every trip where the two differed (8 to 0 and 12 to 0; p = 0.008 and p = 0.0005). The reason is visible in the runs: the robot reached only about 130 of 160 non-repeated hints, against 158 of 160 repeated ones. A spot that one old trip passed is simply not on most routes.

## Two trips, side by side

The same trip, three planners. Left: no hints. Middle: non-repeated hints. Right: the learned repeated states. The red bar is the closed door, the numbered diamonds are the two hints, each line is the robot's recorded path.

<figure>
<video controls muted loop playsinline preload="metadata" poster="{{ '/assets/images/trip_398.png' | relative_url }}">
  <source src="{{ '/assets/video/trip_398.mp4' | relative_url }}" type="video/mp4">
</video>
<figcaption>The plain planner pushes towards the closed bottom door. With non-repeated hints, the robot goes to the first hint, a corner of the top-left room, and stalls there. With the learned checkpoints it passes the left door, then the top door, and reaches the goal at action 167.</figcaption>
</figure>

<figure>
<video controls muted loop playsinline preload="metadata" poster="{{ '/assets/images/trip_325.png' | relative_url }}">
  <source src="{{ '/assets/video/trip_325.mp4' | relative_url }}" type="video/mp4">
</video>
<figcaption>A mirrored trip. Without hints the robot circles at the closed door. The non-repeated hints send it into the corners of the right-hand rooms. The learned checkpoints take it through the right door and the top door to the goal at action 149.</figcaption>
</figure>

I picked these from the trips where only the learned checkpoints succeeded, so they show the failure modes, not a typical trip. On most trips all three versions reach the goal.

## How I got here

The route was not straight, and the earlier post shows the dead ends. The first attempt used door labels from the simulator to decide which states were doors; it worked on one model and not on another. Then I gave the planner hints taken from remembered successful trips similar to the new one. That worked well on both models, but a control showed that *any* point of a matching remembered trip worked as well as the shared point. That was honest but a little disappointing: it meant the benefit came from remembering a matching trip, not from checkpoints.

The step that changed the picture was to stop remembering trips. Once the robot keeps only a small set of learned states and has to reuse them for trips it never made, the repeated states clearly beat the non-repeated ones. One detail mattered: a checkpoint on its own was not enough. My first rule for choosing checkpoints, using the checkpoint states alone, chose the right pair on none of 276 practice trips. Storing the two regions each checkpoint connects fixed that on the first try.

## Where it stands

**Shown, in this world, on two models:**
- The robot learns the repeated states from its own successful experience, with no labels.
- Planning with those 16 states alone cuts failures from about one in five to about one in eighty.
- They help because they are repeated: non-repeated hints of the same kind barely help.

**Not shown yet:**
- **A harder world.** Four rooms have an obvious structure, and a trip needs only two checkpoints. The real test is a bigger world, where trips need several checkpoints in a row and the structure is less obvious.
- **Deeper chains and a hierarchy.** Checkpoints of checkpoints, with the number of levels set by the task.
- **Comparison with other methods,** such as landmark-graph planners.
- **Generality.** Two trained copies of one model, one environment. Small numbers of trips (80 per test), so the exact percentages are uncertain.

## What is next

A bigger world with more rooms and longer routes, then chains of three to five checkpoints, then the first step towards the hierarchy. The code is not published yet; I will add it, with the predeclared rules of every test, when it is ready.
