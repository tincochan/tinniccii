---
layout: post
title: "Building on Top of Black Magic"
description: "Deep generative models are a house of cards built on top of black magic. At least, that’s what I’ve resigned myself to believe."
date: 2023-09-14
tags: [Algorithm]
---

Deep generative models are a house of cards built on top of black magic. At least, that’s what I’ve resigned myself to believe.

<!--more-->

Way back when I started my PhD, generative adversarial networks (GANs), variational autoencoders (VAEs), autoregressive (AR) models, and flow models were the big generative modeling paradigms in town. While each came with its own set of cool mathematical justifications, the degree to which they successfully delivered on the task of human-evaluated sampling quality differed greatly. To this day, it breaks my heart that variational autoencoders (my personal favorite and my area of research during my PhD) had perhaps the *worst* image synthesis performance of the Big Four. This became a topic of consternation for me, and often led me to wonder, *“why are VAEs are so bad at image generation?”* And after years of contemplation, I’ve come to the conclusion:

I don’t know.

It feels pretty defeatist to say that without at least some elaboration, so I’ll pose to you the same hypothetical scenario I had considered myself: imagine if you were transported back to 2014 before deep generative models exploded in popularity and taught the budding ML community the math behind GANs, VAEs, Flows, AR, and diffusion; how would you convince them that, in 2022, the best performing generative model would be diffusion-based models? You traveled back in time with nothing but your mathematical knowledge and you sure as hell were not interested in coding something up in Theano to prove your point. Would you be able to successfully convince them of diffusion models’ practical superiority in image synthesis?

I don’t think I’d be able to. The best I would be able to do is point out potential optimization pit-falls associated with each method, with no conceptual guidelines for when and why one model paradigm should outperform another. In other words, I have no predictive power about any of these models despite years of familiarizing myself with their underlying mathematics. And I suspect I’m not the only one who feels that way. I don’t think the ML community has ever bothered to really come up with an explanation why some model families beat others in practice; we simply shrug our shoulders and attribute it to a case of “you just gotta try everything and see what works”.

As a result of this culture of shrug-hope-and-pray, no one really knows why diffusion models work so well. But *because* it works so well, everyone now builds on top of that black magic by learning about ODEs/SDEs/etc in hopes that the math will suggest what to try next in an effort to squeeze some extra juice out of diffusion models. And this works; the math tells us how to make things more efficient, how to make optimization easier, and suggests certain modifications to try next which, more often than not, lead to modest improvements over the original breakthrough framework. This Magic-Meets-Math industrial complex fuels much of the ML community: people see something that works really well, and then use math to define the search space for what to try next, which makes that something work slightly better. It’s a tried-and-true technique, but one which I think is lacking in introspection and thus stops short of truly explaining existing breakthroughs and predicting new ones. This lack of predictive power is why I think the field of deep generative models, while an engineering and mathematics discipline, is not a scientific discipline quite yet.

Anyway, long story short, I wish we as a field will develop scientific theories for why some deep generative modeling paradigms outperform others in practice. It is by no means an easy task. These models have tons of moving parts and it is unlikely that any one thing is the reason why something works or doesn’t work. I tried during the final years of my PhD to come up with a conceptual justification for why diffusion models outperform VAEs (even though the former is just a special case of the latter[†](https://ruishu.io/2022/10/14/building-on-top-of-black-magic/#fn:j)) and have first-hand experience with how difficult it is to come up with a statement that has sufficient predictive power. I did my best to follow the scientific method and came up with the following:

1. Hypothesis: predictive coding is why diffusion models are better than VAEs at image synthesis
2. Prediction: borrowing the technique of predictive coding and bringing it to VAEs will make VAEs better image synthesizers too.

But I’m glad I tried, and I’m optimistic that there’s more to be learned by forcing our way behind the black magic curtain.
