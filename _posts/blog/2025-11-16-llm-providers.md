---
layout: single
title: "When you say you use an LLM, which model do you mean?"
excerpt: "The models you use have many different lives"
date: 2025-11-15 17:12:54 +0100
categories: blog
permalink: /blog/llm-providers/
classes: wide
comments: false
author_profile: false
share: false
---

I want to make an argument that what you think of as a single LLM actually lives many different lives. You've come to form judgments about each model, and I believe those judgments haven't been entirely fair. By now, you have your favourite models, or at least preferred ones for different use cases. If you're anything like me, you've converged on these preferences through trial and error. Or likely whatever model you were using was good enough that you didn't feel compelled to explore others. Either way, you've formed an opinion about how _good_ each model is.

My argument is that you're likely wrong in your assessment, especially when it comes to open-source models. The reason? You're probably thinking your experience reflects on the _model_ in its pure form, when what you actually interact with is a _system_. This system includes the model plus all the bespoke engineering that supports it: routers, safety layers, sampling parameters, and more. If you're the TL;DR type, that's the gist of it, have a great day.

But if you stick around, I'll show you how this is the case.

## Anthropic's Postmortem

For over a year now, I've seen constant complaints about "they nerfed it" or "they made the model worse", vague accusations that model labs purposefully degraded performance. Usually, you can lump these with your favorite conspiracy theories[^1], harmless speculation that provides entertainment value if nothing else. But a couple of months ago, after more such complaints about Claude showing degraded performance, [Anthropic released a technical report](https://www.anthropic.com/engineering/a-postmortem-of-three-recent-issues) talking of three bugs that may have caused it.

The report is fascinating in its own right, but I want to focus on the first of the three issues they identified: short-context queries (~200K tokens) were being erroneously routed to servers optimized for long-context (~1M tokens). This problem manifested at wildly varying degrees across their three providers (Anthropic's first-party API, Amazon Bedrock, and Google Vertex). Even with all of Anthropic's in-house expertise, when you send a query to Claude, it follows one of three completely different code paths, each with its own optimizations and quirks, producing different results.

This same phenomenon happens with open-source models, except the model labs don't get to control, or even observe it. When you're working with an open-source model, especially if you're hosting it yourself[^2] but also when using third-party inference providers, similar infrastructure problems compound, making your experience of the model worse.

## Moonshot's Vendor Verifier

The speculation about providers serving models at lower precision isn't limited to AI labs. Many neo-clouds and inference providers with their own ASICs seem to serve models that vary wildly in performance. Could they be serving quantized versions without notifying users?[^3] Possibly. But what we know for certain is that performance across different providers varies significantly.

Moonshot, the company behind the Kimi K2 family of models, decided to take matters into their own hands. They tested and [published results](https://github.com/MoonshotAI/K2-Vendor-Verifier) showing how different providers perform when serving the same model. In fact, they updated their results for both [K2](https://github.com/MoonshotAI/K2-Vendor-Verifier?tab=readme-ov-file#k2-0905-evaluation-results) and [K2-thinking](https://github.com/MoonshotAI/K2-Vendor-Verifier?tab=readme-ov-file#k2-thinking--evaluation-results) today.

Their data reveals that some providers lag significantly behind Moonshot's own API, while others perform comparably. What makes this even more challenging is that these numbers aren't static. While things have improved recently (I suspect partly due to efforts like these), previous evals shows similarity scores compared to the official API as low as ~50%.

Imagine if Claude were served by twelve different providers, each doing their own optimizations on their own hardware.

## OpenRouter's Exacto

OpenRouter recently introduced [exacto endpoints](https://openrouter.ai/announcements/provider-variance-introducing-exacto) precisely because different hosts serving the same open-weight model behave differently in practice. The performance spread has tightened over time, see [Artificial Analysis's evaluation](https://artificialanalysis.ai/models/gpt-oss-120b/providers#evaluations) of gpt-oss-120b for reference.

To deliver "what the weights are actually capable of," OpenRouter now routes `model_slug:exacto` (Kimi K2, DeepSeek v3.1 Terminus, GLM4.6, GPT-OSS-120B, Qwen3 Coder) only to providers that simultaneously score near the top on tool-calling accuracy, fall within normal bounds for invoking tools when they're available, and aren't frequently ignored or blacklisted by customers during tool-heavy workloads.

When a marketplace has to create a premium routing tier just to expose the quality in the base weights, you know the market has noticed that performance varies wildly across providers.

## Open Weights vs Closed Source

I argue that your judgment of how good a model is, that mental ranking you've built up, is largely miscalibrated. Or at least misattributed. As we've seen, whether you were using Kimi K2 on their official API or through a third-party provider who hasn't calibrated their systems yet[^4] significantly affected your understanding of how well the Moonshot team did their job. And this bias systematically favors closed-source models.

Once weights are released, the model lab has zero control over how they're served. Closed-source providers control the entire stack. Additionally, open-source models run on everything from consumer GPUs to custom ASICs, with each hardware platform requiring its own optimizations. Meanwhile, if you have Google's full stack abilities (and only Google currently does, though possibly OpenAI in the future), you can actually co-design the architecture and the underlying hardware[^5]. And while this co-design example is the lowest level of optimization and calibration available, there are many layers along the stack that the closed-source providers can control. For non-API products, the difference can be as simple as a different system prompt.

## The Conclusion

I suspect we need to start thinking about "model + provider + runtime config" etc when we are talking about what we are using. This might seem like a silly vocabulary issue, and, I do concede, that to an extent, that is what it is. A user only cares about outcomes, neither the stack that delivers that outcome, nor the specifics of what one called it. But therein lies the issue.

When users encounter provider induced failures, they blame the model, not the infrastructure. This erodes trust in open source models specifically, owing to the high variability, and the overall ecosystem is worse for it. I also think we should demand transparent reporting on quantization settings and behavioral differences.

---

[^1]: I have my own theories about our perception of _degraded_ performance. I don't believe model providers purposefully degrade models by quantizing or throttling them. I believe it's usually a combination of the probabilistic nature of sampling and users becoming more familiar with the system and discovering its rough edges as we delegate more work to it. Remember how magical GPT-3.5 felt initially? Remember how little work you were willing to offload to it? Compare that to GPT-5 or similar models ~3 years later. The same model does not degrade, but your expectations go up.

[^2]: An absolutely valid rebuttal here is: why would you serve these huge, 1T parameter models yourself? While you, as an individual, might not have the need or infrastructure for this challenge, many businesses serve these locally for privacy reasons. These models usually come with MIT or modified MIT licenses that allow it.

[^3]: I would argue this is akin to stealing. Equivalent to buying 1kg of sugar and receiving 800g, even though the package says 1kg.

[^4]: As cited in the Artificial Analysis report, the tweaks required for faithful serving of the weights is something that providers take time to discover for themselves. Model providers seem to help out with most, but with different hardware underneath, it's not entirely a packaged solution.

[^5]: For reference, you can hear Jeff Dean and Noam Shazeer talk to Dwarkesh Patel [here](https://www.dwarkesh.com/p/jeff-dean-and-noam-shazeer) about it. They mention it around the 11-minute mark. The whole conversation is fascinating. I highly recommend it.
