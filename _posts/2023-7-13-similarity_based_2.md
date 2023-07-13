---
layout: post
title: A Better Similarity-Based Bigram Model
---

In my previous post, I discussed the similarity-based bigram model (Dagan et al., 1998) and showed its performance against other classic ngram smoothing techniques. Although the original similarity-based bigram model had less perplexity than the Katz backoff model, it didn't fare as well as the two variations of the Kneser-Ney model on the smaller PTB dataset. In this blog post, I'll introduce a new similarity-based ngram model that surpasses all baseline models.
