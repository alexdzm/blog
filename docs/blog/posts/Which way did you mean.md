---
title: "Which way did you mean"
date: 2024-03-15
author:
  - alexdzm
categories:
  - Mechanistic Interpretability
  - LLMs
slug: getting-started-with-python
description: "A soft intro to modern mechanistic interperatbility"
---


# Which way did you mean?
Like a lot of AI engineers, I have a background in designning and training my own ML models but most of my days are now spent calling an API or running a pretrained models. The differences  between the models I used to work with and the behmouths of state of the art modern LLMs don't stop at how the average engineer uses them, in this post I will explain how I found that even understanding such mod

LLama 3 has just come out and I came across [this work](https://arxiv.org/pdf/2407.01376v1) that shows three effective methods of boosting harm bench scores while maintaing good performance on standard LLM benchmarks. The first two leverage well known parameter efficent fineting techniques (QLoRA &  QLoReFT) to unlearn what the model was taught in fine tuning. In both cases fewer than 20k QA pairs and a single GPU are all that was needed to get the model to tell you how to push your grandmother down the stairs and make it seem like an accident. But much more impressive is the third technique, *refusal orthognalisation*, which requires no fine tuning at all.
 

[Refusaul orthogonalisation](https://arxiv.org/abs/2406.11717), supposes that the ability of a model to refuse to answer a prompt on safety grounds is mediated by a single direction. With the help of thier every slick repo [LINK] and some surprisingly small datasets of harmful + harmless prompts one can find a number of candidate directions for any model. And with ease (and the sheer joy of transformerlens [LINK]) one can run a version of a model where this direction is supressed, the model will do anything.

This whole process caused me to wonder two things Why is this a single direction? and what other concepts in an LLM are mediated by a direction? I was looking to make a model speak french, or in a more formal tone, with no special prompting, only intervention in the latent space
