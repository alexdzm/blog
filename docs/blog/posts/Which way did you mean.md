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
Like a lot of AI engineers, I have a background in designing and training my own ML models but most of my days are now spent calling an API or running pretrained models. The differences between the models I used to work with and the behemoths of state of the art modern LLMs don't stop at thier capabilities or how the average engineer uses them, the way we understand how they work (a field known as [mechanistic interpretability](https://www.neelnanda.io/mechanistic-interpretability/getting-started)) is entirely different. In this post I share my initial investigations into this world, by making good models go bad.

### Jailbreaking LLMs

Llama 3 had just come out and I came across [this work](https://arxiv.org/pdf/2407.01376v1) that shows three effective methods of boosting the likelihood of LLMs returning harmful responses while maintaining good performance on standard LLM benchmarks. The first two leverage well known parameter efficient finetuning techniques (QLoRA & QLoReFT) to unlearn what the model was taught in fine tuning. In both cases fewer than 20k QA pairs and a single GPU are all that was needed to get the model to tell you how to push your grandmother down the stairs and make it seem like an accident. But much more impressive is the third technique, *refusal orthogonalization*, which requires no fine tuning at all.
 
### Refusal orthogonalization 

[Refusal orthogonalization](https://arxiv.org/abs/2406.11717) (RO), supposes that the ability of a model to refuse to answer a prompt on safety grounds is mediated by a single direction. With the help of their very slick repo [LINK] and some surprisingly small datasets of harmful + harmless prompts one can find this direction for many families of models available on hugging face. And with ease one can run a version of a model where this direction is suppressed, the model will do anything. The process of interfering with a model like this to change its behaviors is called model steering.

[OUTPUTS OF MODEL STEERING]

This repo works by taking the mean activations of the model for the harmless and the harmful prompts in the residual stream at every transformer block. These activations are then subtracted from each other, to find the direction that along which they differ. Different directions are selected at each block and each token position after the input.

[insert refusal direction latex]

After this each direction is subtracted in turn from a model with a harmful prompt and the direction that is most effective at suppressing refusal is subtracted.

### Llama 3.2 reproduction

I set about applying this technique to the latest model, released since the paper was announced, [llama 3.2 1B Instruct](https://huggingface.co/meta-llama/Llama-3.2-1B-Instruct). I have updated the main figure from the paper to include this update.

[updated figure]

So this attack is roughly as effective on this new, small, model as it was on its larger slightly older counterparts. This is hardly surprising, this attack vector is not mentioned in the latest llama training paper as something the team were hoping to address.

Note I ran this experiment with Llama guard 2 (as in the original paper) and with the newer Llama guard 3, the results were almost exactly the same. However, I have only included Llama guard 2 results in this post for consistency. 

### Variation

In the refusal direction paper, the authors decide that once a direction is selected in a given layer and position, it should then be ablated in many different places. It is subtracted from the activations at *every* layer, before the transformer block and after the attention heads and MLP. I had been hoping that this approach was a step towards a better understanding of *how* refusal works, but the need to intervene in so many places dashed that hope. In a hope of understanding it better, I experimented with only removing the refusal direction in a subset of these locations (though still at every layer of the model). 
<center>

![Comparison of ablation strategies](../../static/llama_intervention_plot.png)
*Fig 2: Comparison of the refusal and safety scores for llama 3.2 when using different ablation techniques. Full intervention is the combination of the following 3. The intervention locations are shown in Fig 3.*
</center>

These results show that, in the case of Llama 3.2, only ablating this direction in the residual stream is sufficient to recovery the steering behavior that all three interventions produce. This is hardly surprising as the diagram below shows that this single operation is the only one that writes to the residual stream with no skips going around it, in both other cases there remains some contribution to this direction that is able to circumvent being ablated.
<center>

<img src="../../static/llama_diagram.png" alt="llama structure" width="200"/>

*Fig 3: The locations of the different model interventions. 1) Residual Only 2) Attention Only 3) MLP only. Diagram adapted from [llama-2.ai](https://llama-2.ai/llama-2-explained/)*
</center>
This result is not particularly useful for those that want the best refusal suppression, the original technique of ablating everywhere still works and is not computationally expensive at all. On top of this the authors also propose a weight orthogonalization technique which is equivalent and allows for this all to have zero extra cost at inference time over running the model normally.

However this result does suggest an interesting point for those interested in understanding refusal. In this model the attention mechanism does almost nothing to induce refusal (by the mechanism we are suppressing). This suggests that almost the entire refusal direction is learned by the MLP.

I hope to investigate this phenomenon further in other models and by varying how many blocks in the model I intervene on. I am intrigued to see if this can be linked with [this recent work](https://arxiv.org/abs/2406.15786) which found that a lot of attention layers can be removed from a trained model without degrading performance.

### What next?

RO is a relatively simple interpretability experiment for LLMs. I am now spending time investigating sparse auto encoders (SAEs) with a particular focus on the [Gemma Scope](https://ai.google.dev/gemma/docs/gemma_scope) models that have been publicly released. I am curious to find out if these complex features can be steered in the same way as refusal can.