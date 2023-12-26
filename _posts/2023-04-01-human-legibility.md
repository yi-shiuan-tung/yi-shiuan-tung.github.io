---
layout: post
title: Improving Human Legibility in Collaborative Robot Tasks through Augmented Reality and Workspace Preparation
description: Improve human legibility by projecting virtual obstacles and rearranging objects in the workspace.
date: 2023-04-01
tags: publications
---

## Motivation

The canonical approach for human robot interaction is to first predict the human motion and then generate a robot plan. This requires a robust and accurate predictions of human behavior. If the human models are inaccurate, the robot may produce unsafe interactions.

To achieve more fluent human robot interaction, there has also been work on improving the robot’s expressiveness of its intentions. Dragan et al. developed a formalism for legibility which is the probability of successfully predicting an agent’s goal given an observation of a snippet of its trajectory [^dragan_legibility]. Subsequent works have shown that robots with legible movements lead to greater task efficiency, trustworthiness, and sense of safety in human robot collaboration. However, there hasn’t been work on looking at how human legibility can improve the robot’s ability to collaborate with humans.

## Our Approach

We improve the robot's prediction of the human’s goal (aka the human’s legibility) by **projecting virtual barriers in augmented reality and rearranging objects in the workspace**. We present a metric that evaluates an environment configuration in terms of its legibility and present an optimization approach for autonomously generating object and virtual barrier placements.

<center>
<img src="/blog/assets/img/human_legibility/legible_reach.jpg" alt="Tabletop Manipulation Task" width="300" style="margin-right:30px">
<img src="/blog/assets/img/human_legibility/navigation_exp.jpg" alt="Warehouse Navigation Task" width="310">
</center>

<br>

## Method

### Legibility Metric

First we define the legibility metric that scores an environment configuration in terms of how legibile it is to approach a goal.

<center>
<img src="/blog/assets/img/human_legibility/env_leg_gt.png">
</center>

Given a ground truth goal $G_{true}$ (blue circle in the example below), we generate a snippet of the human's trajectory approaching the blue circle. In our experiments we use [visibility graphs](https://en.wikipedia.org/wiki/Visibility_graph). If the most likely goal is not the blue circle, we penalize by a fixed cost $c$. 

<center>
<img src="/blog/assets/img/human_legibility/env_leg_fixed_cost.png" width="500">
</center>

If the robot is able to correctly predict the ground truth goal, we still want to award configurations with more confident predictions. Therefore, the score is the margin function which is the difference between the most likely and second most likely goal probabilities.

<center>
<img src="/blog/assets/img/human_legibility/margin.png" width="380">
</center>

<br>

### Legibility Optimization

Now that we can evaluate the legibility of an environment configuration given a ground truth goal, we present an objective function that evaluates the legibility for a task.

<br>

<center>
<img src="/blog/assets/img/human_legibility/leg_opt.png" width="500">
</center>

<br>

We assume the task $T$ has general precedence constraints. A subtask can only begin after all of its precedence constraints are completed. For example, In an assembly task, you need to finish making the drawers before inserting them into their final positions. In the objective function, we iterate over all possible valid orderings of task $T$, which are the orderings that satisfy precedence constraints. We find the possible ground truth goals at a given state of a task and sum up the legibility scores (EnvLegibility).

### Quality Diversity for Efficient Search

Now that we know how to evaluate the legibility of a given workspace configuration, we need to search through the space of possible configurations to find the most legible one. This is computationally intractable for most non-trivial applications. We use a quality diversity approach called MAP-Elites [^map_elites] to efficiently search through the space of possible workspace configurations.

The MAP-Elites algorithm consists of two phases, the initialization phase and the improvement phase. In the initialization phase, a random environment is generated and placed in a grid where the dimensions of the grid are some features or behaviors of interest. Here we use the ordering of the cubes in the x-axis and the min distance between cubes. If two environments have the same cell in the grid, the one that has a higher legibility score is stored.

<center>
<img src="/blog/assets/img/human_legibility/map_elites_init.png" width="300">
<figcaption>MAP-Elites Initialization</figcaption>
</center>

<br>

In the improvement phase, we sample a cell from the grid and run gradient descent. In gradient descent, we first sample new positions for each cube from a Gaussian centered at the cube’s current position. The new configuration may end up in a different cell in the grid. If the cell is not populated or if the new configuration has a better legibility score, then the new configuration is stored in the cell. Then, two cubes are randomly selected and a virtual obstacle of a fixed size is placed between them. We store the new configuration if the legibility score is better.

<center>
<img src="/blog/assets/img/human_legibility/map_elites_improve.png" width="300">
<figcaption>MAP-Elites Improvement Phase</figcaption>
</center>

<br>

Below is a gif of the gradient descent process.
<center>
<img src="/blog/assets/img/human_legibility/gradient_descent.gif" width="300">
</center>


## Demo

<iframe
    width="840"
    height="480"
    src="https://youtube.com/embed/QVV9SW4lyns"
    frameborder="0"
    allow="autoplay"
    allowfullscreen
>
</iframe>

<br>
<br>

[^dragan_legibility]: A. D. Dragan, K. C. T. Lee and S. S. Srinivasa, "Legibility and predictability of robot motion," 2013 8th ACM/IEEE International Conference on Human-Robot Interaction (HRI), Tokyo, Japan, 2013, pp. 301-308, doi: 10.1109/HRI.2013.6483603.
[^map_elites]: Mouret, Jean-Baptiste, and Jeff Clune. "Illuminating search spaces by mapping elites." arXiv preprint arXiv:1504.04909 (2015).