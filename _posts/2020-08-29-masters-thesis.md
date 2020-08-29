---
layout: post
title:  "Master's Thesis: Probabilistic Regression using Conditional Generative Adversarial Networks"
date:   2020-08-29 22:48:00 +0200
author: Joel Oskarsson
categories: ai, machine learning, GAN
---

I have finished my master's thesis: [Probabilistic Regression using Conditional Generative Adversarial Networks](http://urn.kb.se/resolve?urn=urn:nbn:se:liu:diva-166637).

### Abstract
Regression is a central problem in statistics and machine learning with applications everywhere in science and technology. In probabilistic regression the relationship between a set of features and a real-valued target variable is modelled as a conditional probability distribution. There are cases where this distribution is very complex and not properly captured by simple approximations, such as assuming a normal distribution. This thesis investigates how conditional Generative Adversarial Networks (GANs) can be used to properly capture more complex conditional distributions. GANs have seen great success in generating complex high-dimensional data, but less work has been done on their use for regression problems. This thesis presents experiments to better understand how conditional GANs can be used in probabilistic regression. Different versions of GANs are extended to the conditional case and evaluated on synthetic and real datasets. It is shown that conditional GANs can learn to estimate a wide range of different distributions and be competitive with existing probabilistic regression models.

Code is available [here](https://github.com/joeloskarsson/CGAN-regression).

