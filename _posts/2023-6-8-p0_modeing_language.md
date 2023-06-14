---
layout: post
title: Modeling Language 0 Introduction
---

How do we acquire and use language? how do babies learn word meanings, word compositions and grammar? how can children produce phrases or sentences that they themselves have never heard? and how can a computer do the same? 

People have been curious about the mystery of language, especially its relation to human intelligence, for many decades, and there are a great number of research trying to seek answer to these questions scientifically through various different perspectives. Great amount of progress has been made, many theories have been proposed, but many of these questions still remain open. We're closer to the truth, but still not there yet.

In this series of blog posts, I will try to answer some of these questions from what I've read, what I've thought and what I've observed, with a heavy focus on the "computational modeling" side. 

What is modeling? in my opinion it's simply the act of replacing the subject of interest with something else that is similar in quality, appearance or behavior, but smaller, cheaper, more manageable and controllable. For example, we may use aircraft models to study the aerodynamics of real airplanes, use weather models to predict the future weather, or use fashion models and mannequins to display how a piece clothing may look on customers' body. Similarly, we can use a **model of the language** to study human language, cognition and intelligence. Importantly, modeling is a cyclic process: we use our initial theory to build a model, we use the model to test the hypotheses and propose better theories, and use that to design better models. 

<img class="centered bg-white" src="https://raw.githubusercontent.com/DeMoriarty/DeMoriarty.github.io/master/images/modelig_cycle.png"/>  

It may sound like this series will be all about Large Language Models (LLM), since that's probably the first thing come to our mind when talking about modeling language. LLMs are extremely good at modeling language, we must give them credit. But how much can they contribute to understanding of language and human mind? Well, not a lot. One of the biggest problems of LLMs is that they are huge black-boxes, which means they are not inherently interpretable. Remember that modeling is a cyclic process, when modeling doesn't provide better understanding, we can't make a better model. In fact, most of the recent advancement in LLMs comes from one ridiculously simple idea: bigger is better. 

Our brain relies on statistics when learning a language. And statistical learning should be one of the main building block of any linguistic theory. But that doesn't justify the absence of interpretability. I believe that a statistical model of language can totally be interpretable, as long as it's built gradually within the modeling-understanding cycle.
