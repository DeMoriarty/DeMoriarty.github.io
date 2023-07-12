---
layout: post
title: Similarity-based Generalization in N-gram Models (unfinished)
---

If you're new to N-gram models, I recommend starting with my [introduction to N-gram language models](../ngram_intro).

Using a finite set of words, we as humans can produce infinite number of phrases and sentences. As we increase N, the number of possible N-grams grows exponentially, but the number of N-grams we can observe from a limited text corpus is always proportional to the corpus's size. This leads to what's known as the data sparsity problem, a fundamental issue in N-gram language modeling. In my previous post, I discussed various approaches to address this problem. I believe that the most crucial strategy among these is *Similarity-based Generalization*.

The Similarity-based Generalization approach was initially suggested for bigram models. The idea is straightforward: if two words are semantically similar, the words that follow them should also have a similar distribution. Here's a simple example: if "orange", "lemon", and "grapefruit" are similar, and if "orange juice" and "lemon juice" are present in the training data, but "grapefruit juice" is not, we can estimate the likelihood of "juice" following "grapefruit" using the probabilities of "juice" coming after "orange" and "lemon".

An interesting question here is how we decide which words are similar. The answer lies in distributional similarity. To understand what this means, I recommend checking out my [introduction to Distributional Semantics](../distributional_semantics).

## The case for bigrams
The similarity-based bigram model (Dagan et al., 1998) has three key components:

1. $$S(w_1)$$: a set of words that are similar to the context word $$w_1$$. $$S(w_1)$$ contains $$k$$ elements, where $$k$$ is a hyperparameter; 
2. $$W(w_1^ \prime, w_1)$$: the rescaled (dis)similarity score between $$w_1$$ and  $$w_1^ \prime$$, where $$w_1^ \prime$$ is a member of $$S(w_1)$$;
3. $$P_{MLE}(w_2 \mid w_1)$$: the Maximum Likelihood probability estimation of $$w_2$$ occurring after $w_1$;
4. $$P_{SIM}(w_2 \mid w_1)$$: an interpolation model that mixes the $$P_{MLE}$$ of the next word given each word in $$S(w_1)$$ as context;
5. $$P_d(w_2 \mid w_1)$$: a discounted bigram model such as the Good-Turing model;
6. $$P_r(w_2 \mid w_1)$$: an interpolation model that mixes a unigram model $$P_{MLE}(w_2)$$ with $$P_{SIM}$$.

At the very top, the similarity-based bigram model is simply a backoff model, similar to the Katz backoff (Katz, 1987). In a backoff model, when a given bigram is seen in the corpus, it simply uses the ML probabilities, but when the bigram is unseen, it uses the probability estimates from a supplementary model. But unlike Katz, where the supplementary model is a lower order model, the similarity-based bigram model backs off to another interpolation model ($$P_r$$):

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

The interpolation model $$P_{r}$$ is basically a mixture of a unigram model $$P_{MLE}(w_2)$$ and a similarity-based model $$P_{SIM}(w_2 \mid w_1)$$, where the interpolation is controlled by a hyperparameter $$\gamma$$:

$$
\begin{equation}
	P_{r}(w_2 \mid w_1) = \gamma P_{MLE}(w_2) + (1 - \gamma)P_{SIM}(w_2 \mid w_1)
\end{equation}
$$

$$P_{SIM}(w_2 \mid w_1)$$ is computed by averaging $$P_{MLE}(w_2 \mid w_1^\prime)$$ of every $$w_1^\prime$ in $$S(w_1)$$, with weights determined by rescaling and normalizing the similarity scores of $$w_1^\prime$$ and $$w_1$$:

$$
\begin{equation}
	P_{SIM}(w_2 \mid w_1) = \sum_{w_1^ \prime \in S(w_1)}P_{MLE}(w_2 \mid w_1^ \prime)\dfrac{W(w_1, w_1^ \prime)}{\sum_{w_1^ \prime \in S(w_1)}W(w_1, w_1^ \prime)}
\end{equation}
$$

$$P_d(w_2 \mid w_1)$$ is a discounted bigram model, and similar to Katz, the Good-Turing model was chosen. In the Good-Turing model, the actual counts of bigrams are replaced with adjusted counts during the probability estimation:

$$
\begin{equation}
	P_d(w_2 \mid w_1) = \dfrac{
		C^{*}(w_1, w_2)
	}{
		C(w_1)
	}
\end{equation}
$$

The calculation of the adjusted count $$C^{*}$$ involves a unique concept: frequency of frequencies. $$N_c$$ represents the number of unique bigrams in the corpus that have frequency $$c$$. $$C^{*}$$ is computed as:

$$
\begin{equation}
	C^{*}(w_1, w_2) = 
	(C(w_1, w_2) + 1)
	\dfrac{
		N_{C(w_1, w_2) + 1}
	}{
		N_{C(w_1, w_2)}
	}
\end{equation}
$$

The parameter $$\alpha$$ used in the calculation of $$\hat{P}(w_2 \mid w_1)$$ is obtained using the following formula:

$$
\begin{equation}
\alpha(w_1) = \dfrac{
	1 - \sum_{w_2 : C(w_1, w_2) > 0} P_d(w_2 \mid w_1)
}{
	\sum_{w_2 : C(w_1, w_2) =>= 0} P_r(w_2 \mid w_1)
}
\end{equation}
$$

The choice of the similarity measure and rescaling function is another important design decision. The authors used negative exponential KL divergence in their language modeling experiments:

$$
D( w_1 \parallel w_1^ \prime ) = \sum_{w_2 \in V }
P(w_2 \mid w_1) \log 
\dfrac
{
	P(w_2 \mid w_1)
}
{
	P(w_2 \mid w_1 ^ \prime)
}
$$

$$
W(w_1, w_1 ^ \prime) = 10 ^ {-\beta D(w_1 \parallel w_1^ \prime) }
$$

here, $$V$$ refers the vocabulary, which is the collection of all unique words in the text corpus, and $$\beta$$ is another hyperparameter of the model.

### Selecting the Similarity Metric
In addition to the negative exponential of KL divergence used in the original paper, we will also explore the use of cosine similarity with two different distributional modeling approaches.

$$
\mathrm{sim}(a, b) = \dfrac{
	\vec{v}_a \cdot \vec{v}_b
}{
	\| \vec{v}_a \| \| \vec{v}_b \|
}
$$

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

In their experiments, the original model achieved 20% improvement in perplexity over Katz backoff model. However, in the following experiments, we will see that if we discard the backoff model, and only use the interpolation model $P_r$ as the final estimation, the performance of the model will be even better.

## Bigram Language Modeling Experiments
Let's test out the above mentioned approaches with some language modeling experiments on Penn treebank (PTB) and wikitext103 datasets. PTB contains around a million words with trainset and test set combined, while Wikitext103 is roughly 100 times larger than that. We will use a vocabulary size of 5000 for both datasets, containing most frequent words. 

### baselines
1. Laplace smoothing:

$$
\begin{equation}
	P_d(w_2 \mid w_1) = \dfrac{
		C(w_1, w_2) + 1
	}{
		C(w_1)
	}
\end{equation}
$$

2. Good-Turing:
Originally proposed in [this paper](https://www.jstor.org/stable/2333344) and explained in [these slides](https://web.stanford.edu/class/archive/cs/cs224n/cs224n.1086/handouts/cs224n-lecture2-language-models-slides.pdf).

3. Katz backoff:
Originally proposed in [this paper](https://www.researchgate.net/publication/2572004_Estimation_of_probabilities_from_Sparse_data_for_the_language_model_component_of_a_speech_recognizer) and explained in [these slides](https://web.stanford.edu/class/archive/cs/cs224n/cs224n.1086/handouts/cs224n-lecture2-language-models-slides.pdf).

4. Interpolated Kneser-Ney:
Originally proposed in [this paper](https://www-i6.informatik.rwth-aachen.de/publications/download/951/Kneser-ICASSP-1995.pdf) and explained in [these slides](https://www.cse.iitb.ac.in/~pjyothi/cs753/slides/lecture10.pdf) and [this paper](https://arxiv.org/pdf/cs/0108005.pdf).

5. Modified Kneser-Ney:
Originally proposed in [this paper](https://people.eecs.berkeley.edu/~klein/cs294-5/chen_goodman.pdf) and explained in [this blog post](http://www.foldl.me/2014/kneser-ney-smoothing/).



