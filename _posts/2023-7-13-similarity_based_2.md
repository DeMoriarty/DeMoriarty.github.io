---
layout: post
title: An Imporved Similarity-Based Bigram Model (unfinished)
---

In [my previous post](../similarity_based), I discussed the similarity-based bigram model (Dagan et al., 1998) and showed its performance against other classic ngram smoothing techniques. Although the original similarity-based bigram model had less perplexity than the Katz backoff model, it didn't fare as well as the two variations of the Kneser-Ney model on the smaller PTB dataset. In this blog post, I'll introduce a new similarity-based bigram model that surpasses all previous models.

## An Enhanced Model

This enhanced bigram model, based on Dagan et al's work, includes several evidence-based tweaks. The first notable change is eliminating backoff. The final probability estimate, represented as $$\hat{P}$$, now stems directly from $$P_r$$. This is a blend of a raw similarity-based estimate, $$P_{SIM}$$, and a complementary model $$P_c$$. The weightage of each part in the final estimate is controlled by an interpolation parameter, $$\lambda$$. This is a variable value between $$[0, 1]$$, and can be set using a hyperparameter search procedure, like grid search.

$$
\hat{P}(w_2 \mid w_1) = P_{r}(w_2 \mid w_1) = \gamma P_c(w_2 \mid w_1) + (1 - \gamma)P_{SIM}(w_2 \mid w_1)
$$

In the older model, $$P_{SIM}$$ was calculated by averaging maximum likelihood estimates of the next word probability for each word in $$S(w_1)$$. But in this new model, similar words in $$S(w_1)$$ are used to estimate bigram counts, and then $$P_{SIM}$$ is derived by normalizing these estimates.

$$
\tilde{C}(w_1, w_2) = \sum_{w_1^ \prime \in S(w_1)}
C(w_1^ \prime, w_2)
\dfrac{
	W(w_1^ \prime, w_1)
}{
	\sum_{w_1^ \prime \in S(w_1)}W(w_1^ \prime, w_1)
}
$$

$$
P_{SIM}(w_2 \mid w_1) = 
\dfrac
{\tilde{C}(w_2, w_1)}
{\sum_{w_1^ \prime \in S(w_1)} \tilde{C}(w_2, w_1^\prime)}
$$

$$W(w_1^ \prime, w_1)$$ stands for the adjusted similarity score. In the new model, we use cosine similarity as the similarity measure, given its effectiveness in distributional semantics.

$$
D(w_1, w_1 ^ \prime) = 
\dfrac
{
	\vec{v}_{w_1} \cdot \vec{v}_{w_1 ^ \prime}
}
{ 
	\| \vec{v}_{w_1} \|  \| \vec{v}_{w_1 ^ \prime} \| 
}
$$

Cosine similarity scores are then adjusted with parameter $$\beta$$ and raised to the power of e.

$$
W(w_1, w_1 ^ \prime) = e^{ \beta D(w_1, w_1 ^ \prime )}
$$

It's interesting to mention that the exponential function paired with the normalization of similarity scores during $$\tilde{C}$$ calculation, is essentially the same as the softmax function with temperature.

$$
\dfrac{
	e^{ \beta W(w_1, w_1^ \prime) }
}{
	\sum_{w_1^ \prime \in S(w_1)} e^{ \beta W(w_1, w_1^ \prime) }
}
$$

$$\vec{v}_{w_1}$$ represents the distributional vector representation of $$w_1$$. We'll explore two possible distributional methods: 

**Log of Laplace**

$$
(\vec{v}_a)_i = \log( C(a, V_i) + 1)
$$

**Positive pointwise mutual information (PPMI)**

$$
(\vec{v}_a)_i = 
\max\left(0,
\log 
\dfrac{
	P_{MLE}(a, V_i)
}{
	P_{MLE}(a) P_{MLE}(V_i)
}
\right)
$$

Here, $$(\vec{v}_a)_i$$ refers to the i'th element of the vector $$\vec{v}_a$$, and $$V_i$$ refers to the i'th word in the vocabulary.

The complementary model, $P_c$, can be picked from the existing n-gram methods. We will consider the unigram model, Katz backoff model, and the interpolated Kneser-Ney model as possible choices for the complementary model.
