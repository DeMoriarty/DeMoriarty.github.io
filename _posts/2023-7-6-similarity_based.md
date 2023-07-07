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

1. $$S(w_1)$$: a set of words that are similar to the context word $$w_1$$; 
2. $$W(w_1^ \prime, w_1)$$: the rescaled similarity score between $$w_1$$ and  $$w_1^ \prime$$, where $$w_1^ \prime$$ is a member of $$S(w_1)$$;
3. $$P_{MLE}( w_2 \mid w_1 )$$ and $$P_{MLE}( w_2 )$$: the Maximum Likelihood Estimation (MLE) of the probability of $$w_2$$ given $$w_1$$, as well as the unigram probability of $$w_2$$

The final probability estimate ($$P_{r}$$) comes from the interpolation of a unigram model $$P_{MLE}(w_2)$$ and a similarity-based model $$P_{SIM}(w_2 \mid w_1)$$, where the interpolation is controlled by a hyperparameter $$\gamma$$ . $$P_{SIM}(w_2 \mid w_1)$$ is computed by averaging $$P_{MLE}(w_2 \mid w_1^\prime)$$ of every $$w_1^\prime$$ in $$S(w_1)$$, with weights determined by rescaling and normalizing the similarity scores of $$w_1^\prime$$ and $$w_1$$.

$$
P_{SIM}(w_2 \mid w_1) = \sum_{w_1^ \prime \in S(w_1)}P_{MLE}(w_2 \mid w_1^ \prime)\dfrac{W(w_1, w_1^ \prime)}{\sum_{w_1^ \prime \in S(w_1)}W(w_1, w_1^ \prime)}
$$

$$
P_{r}(w_2 \mid w_1) = \gamma P_{MLE}(w_2) + (1 - \gamma)P_{SIM}(w_2 \mid w_1)
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

here, $$V$$ refers the vocabulary, which is the collection of all unique words in the text corpus, and $\beta$ is another hyperparameter of the model.

In their experiments, this model achieved 20% improvement in perplexity over Katz backoff model (Katz, 1987). 

## Extending to higher order N-grams

The interpolative nature of the bigram model can easily be extended to arbitrary order N-grams in a recursive manner. The N-gram model's final probability estimation ($$P_r$$) is an interpolation between the similarity-based model ($$P_{SIM}$$) of the current order and the estimated probability ($P_r$) of the preceding (N-1)th order. In the base case of a unigram model, $$P_r$$ is simply equal to the Maximum Likelihood Estimation ($$P_{MLE}$$):

$$P_{r}(w_i \mid w_{i-n+1 : i-1}) = \gamma P_r(w_i \mid w_{i-n+2 : i-1}) + (1-\gamma)P_{SIM}(w_i \mid w_{i-n+1 : i-1})$$

$$P_{r}(w_i) = P_{MLE}(w_i)$$

The main challenge lies in the calculation of $$P_{SIM}$$. As a first step, we could extend the equation for bigrams to N-grams:

$$
\begin{gather}
	P_{SIM}(w_{i} \mid w_{i-n+1 : i-1}) 
	= \sum_{w_{i-n+1 : i-1}^ \prime \in S(w_{i-n+1 : i-1})}P_{MLE}(w_{i} \mid w_{i-n+1 : i-1}^ \prime)
	\dfrac {W(w_{i-n+1 : i-1}^ \prime, w_{i-n+1 : i-1})}
	{\sum_{w_{i-n+1 : i-1}^ \prime \in S(w_{i-n+1 : i-1})}W(w_{i-n+1 : i-1}^ \prime, w_{i-n+1 : i-1})}
\end{gather}
$$

But functions like $$S(w_{i-n+1 : i-1})$$ and $$W(w_{i-n+1 : i-1}^ \prime, w_{i-n+1 : i-1})$$ need us to clearly define what it means for two N-grams to be similar.  This plunges us into the deep end of compositionality. Here, I propose three candidates:

### 1. ### Possible Definitions of N-gram Similarity
**Mean of Constituents**
As the name suggests, the similarity of two N-grams is defined as the average distributional similarity of each pair of words in both N-grams. For example:

$$
\mathrm{sim}( \texttt{good man}, \texttt{great guy} ) = 
\dfrac{
	\mathrm{sim}( \texttt{good}, \texttt{great}) + \mathrm{sim}( \texttt{man}, \texttt{guy})
}
{2}
$$

This works well in some cases, but not when N-grams are non-compositional. For instance, it fails to capture the similarity between $$\texttt{best man}$$ and $$\texttt{groom's person}$$, because $$\texttt{best}$$ and $$\texttt{groom's}$$ aren't really synonymous.

**Additive Composition and Multiplicative Composition**
Additive or multiplicative combination of vectors is a simple yet effective method in Compositional Distributional Semantics. The suggestion is that the vector representation of a phrase can be created by merely adding or multiplying the vectors of its individual words. Using this method, we can create vector representations for each N-gram and simply treat them as word vectors. For example:

$$
\mathrm{sim}( \texttt{good man}, \texttt{great guy} ) = 
\mathrm{sim}( \texttt{good} + \texttt{man}, \texttt{great} + \texttt{guy} )
$$

or 

$$
\mathrm{sim}( \texttt{good man}, \texttt{great guy} ) = 
\mathrm{sim}( \texttt{good} \odot \texttt{man}, \texttt{great} \odot \texttt{guy} )
$$

The main issue with this method is that it disregards word order. For instance, $$\texttt{car company}$$ and $$\texttt{company car}$$ would have identical representations. Moreover, like the Mean of Constituent, Additive Composition also fails with non-compositional examples.

**N-grams as Words**
This approach is in stark contrast to the previous two. It essentially treats all N-grams as words, and directly apply distributional semantics approaches to N-grams, while ignoring all information from their constituents. In theory, using this method, N-grams can be directly compared to words or N-grams of any size.

$$
\begin{align}
	&\mathrm{sim}( \texttt{good man}, \texttt{great guy} ) = \mathrm{sim}( \texttt{good\\_man}, \texttt{great\_guy} )
\\
	&\mathrm{sim}( \texttt{united kingdom}, \texttt{Britain} ) = \mathrm{sim}( \texttt{united_kingdom}, \texttt{Britain} )
	\end{align}
$$

The weakness of this approach is rather obvious: it ignores compositionality altogether, and it is susceptible to data sparsity. 

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

to be continued...
