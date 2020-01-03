---
layout: post
title:  "Gridworld Sandbox for Reinforcement Learning"
date:   2019-02-24 11:05:00 +0200
author: Joel Oskarsson
categories: ai, machine learning, reinforcement learning
---

I have always found that one of the best ways to truly understand a concept is to implement it in code. So when I got a couple days off in between my exams and the spring semester I decided to revisit one of the most interesting topics I studied this fall, but never really had the time to dive deep into amidst term exams and project deadlines. That is the topic of reinforcement learning, or more specifically the [model-free](https://en.wikipedia.org/wiki/Model-free_(reinforcement_learning)) approach of [Q-learning](https://en.wikipedia.org/wiki/Q-learning).

This project is a small application written in Python 3 that simulates a gridworld environment and an agent that can be trained using Q-learning. All code can be found on [github](https://github.com/joeloskarsson/rl-gridworld). The environment and learning hyperparameters can be adjusted in the GUI. Training can be followed in real time as the agent moves around the environment. Once a good estimate of the Q-function has been learned an optimal policy w.r.t. the learned function can be used.

<div style="text-align: center; margin: 20px 0px;">
    <img src="https://raw.githubusercontent.com/joeloskarsson/rl-gridworld/master/screenshots/screenshot0.png" width="800px" style="margin: auto">
    <span>Screenshot of gridworld environment</span>
</div>

The GUI was written in [PyQt](https://wiki.python.org/moin/PyQt), a library that I haven't worked with before. I have some bad experiences with python GUI libraries (read Tkinter), but I have to say that working with PyQt was really smooth. Qt is a powerful toolkit and PyQt has shown to be a quite accessible binding. I had a really easy time to get up and running and make the GUI look how I wanted, so PyQt will definitely be my go to Python GUI library for future projects.

My continued exploration of the area of reinforcement learning has led me more into deep RL. I found a great resource in Open Ai's "[Spinning Up in Deep RL](https://spinningup.openai.com/en/latest/index.html)" that I look forward to explore.
