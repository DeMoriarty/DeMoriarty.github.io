---
layout: post
title: An Imporved Similarity-Based Bigram Model
---

In my previous post, I discussed the similarity-based bigram model (Dagan et al., 1998) and showed its performance against other classic ngram smoothing techniques. Although the original similarity-based bigram model had less perplexity than the Katz backoff model, it didn't fare as well as the two variations of the Kneser-Ney model on the smaller PTB dataset. In this blog post, I'll introduce a new similarity-based bigram model that surpasses all previous models.

## An Improved Model
### **Discarding backoff**:
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

### **Different distributional similarity methods**
In addition to the negative exponential of KL divergence used in the original paper, we will also explore the use of cosine similarity with following two distributional modeling methods:

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

