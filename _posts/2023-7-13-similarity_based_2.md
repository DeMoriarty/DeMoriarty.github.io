---
layout: post
title: An Imporved Similarity-Based Bigram Model
---

In [my previous post](../similarity_based), I discussed the similarity-based bigram model (Dagan et al., 1998) and showed its performance against other classic ngram smoothing techniques. Although the original similarity-based bigram model had less perplexity than the Katz backoff model, it didn't fare as well as the two variations of the Kneser-Ney model on the smaller PTB dataset. In this blog post, I'll introduce a new similarity-based bigram model that surpasses all previous models.

## An Improved Model

The following changes are made to the original similarity-based bigram model.

### Discarding backoff:

The original model was a backoff model at the very top level:

$$
\begin{equation}
	\hat{P}(w_2 \mid w_1) = 
	\left\{
	\begin{array}{ll}
		P_d(w_2 \mid w_1) &\mathrm{if} \; C(w_1, w_2) > 0  
		\\
		\alpha(w_1) P_r(w_2 \mid w_1) &\mathrm{otherwise}
	\end{array}
	\right.
\end{equation}
$$

But now we'll be simply using $$P_r$$ as the final probability estimate:

$$
\hat{P}(w_2 \mid w_1) = P_r(w_2 \mid w_1)
$$

### Different distributional similarity methods

Instead of the negative exponential of KL divergence used in the original paper, we will explore the use of cosine similarity with following two distributional modeling methods:

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

Here, $$(\vec{v}_a)_i$$ denotes the i'th element of the vector $$\vec{v}_a$$, and similarly $$V_i$$ refers to the i'th word in the vocabulary.

Cosine similarity is calculated as follows:

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

Because the similarity metric has changed to cosine similarity, the rescale function $$W$$ should also change correspondingly. Exponential function is chosen in the improved model. 

$$
\begin{equation}
	W(w_1, w_1 ^ \prime) = e^{ \beta D(w_1, w_1 ^ \prime )}
\end{equation}
$$

Note that, in combination with the normalization during the calculation of $P_{SIM}$, it becomes equivalent to softmax with temperature.

### Using similar words to estimate counts instead of probabilties

Originally, $$P_{SIM}$$ was calculated as the weighted average of the $$P_{MLE}$$ of each similar word.

$$
\begin{equation}
	P_{SIM}(w_2 \mid w_1) = \sum_{w_1^ \prime \in S(w_1)}P_{MLE}(w_2 \mid w_1^ \prime)\dfrac{W(w_1, w_1^ \prime)}{\sum_{w_1^ \prime \in S(w_1)}W(w_1, w_1^ \prime)}
\end{equation}
$$

Instead, now we will use similar words to estimate counts, rather than probabilities.

$$
\tilde{C}(w_2 w_1) = \sum_{w_1^ \prime \in S(w_1)}
C(w_2 w_1^ \prime)
\dfrac{
	W(w_1^ \prime, w_1)
}{
	\sum_{w_1^ \prime \in S(w_1)}W(w_1^ \prime, w_1)
}
$$

$$
P_{SIM}(w_2 \mid w_1) = 
\dfrac
{\tilde{C}(w_2 w_1)}
{\sum_{w_1^ \prime \in S(w_1)} \tilde{C}(w_2 w_1^\prime)}
$$

### Dynamic calculation of the interpolation parameter $$\gamma$$

$$\gamma$$ is a parameter that's used to determine the contribution of the unigram and bigram models during the calculation $$P_r$$. It's a free parameter of the model that's chosen through some form of search.

$$
\begin{equation}
	P_{r}(w_2 \mid w_1) = \gamma P_{MLE}(w_2) + (1 - \gamma)P_{SIM}(w_2 \mid w_1)
\end{equation}
$$

In the improved model, $$\gamma$$ is dynamically calculated using $$C$$:

$$
\gamma = \dfrac{1}{
	\alpha C(w_1, w_2) + 1
}
$$

This introduces another free parameter $$\alpha$$.

### The final model

1. 

$$
\begin{equation}
	\hat{P}(w_2 \mid w_1) = P_{r}(w_2 \mid w_1) = \gamma P_{MLE}(w_2) + (1 - \gamma)P_{SIM}(w_2 \mid w_1)
\end{equation}
$$

2.
$$
\gamma = \dfrac{1}{
	\alpha C(w_1, w_2) + 1
}
$$

3.

$$
P_{SIM}(w_2 \mid w_1) = 
\dfrac
{\tilde{C}(w_2 w_1)}
{\sum_{w_1^ \prime \in S(w_1)} \tilde{C}(w_2 w_1^\prime)}
$$

4.

$$
\tilde{C}(w_2 w_1) = \sum_{w_1^ \prime \in S(w_1)}
C(w_2 w_1^ \prime)
\dfrac{
	W(w_1^ \prime, w_1)
}{
	\sum_{w_1^ \prime \in S(w_1)}W(w_1^ \prime, w_1)
}
$$

5.

$$
\begin{equation}
	W(w_1, w_1 ^ \prime) = e^{ \beta D(w_1, w_1 ^ \prime )}
\end{equation}
$$

6.

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

7.



