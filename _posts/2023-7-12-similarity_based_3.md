---
layout: post
title: Extending the Similarity-Based Model to Higher Order N-grams (unfinished)
---

This is the continuation of [Similarity-based Generalization in N-gram Models](..similarity_based).

## Extending to higher-order N-grams
The interpolative nature of the bigram model can easily be extended to arbitrary order N-grams in a recursive manner. The N-gram model's final probability estimation ($$P_r$$) is an interpolation between the similarity-based estimate ($$P_{SIM}$$) of the current order and the estimated probability ($$P_r$$) of the preceding (N-1)th order. In the base case of a unigram model, $$P_r$$ is simply equal to the Maximum Likelihood Estimation ($$P_{MLE}$$):

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

### Possible Definitions of N-gram Similarity
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
	&\mathrm{sim}( \texttt{good man}, \texttt{great guy} ) = \mathrm{sim}( \texttt{good_man}, \texttt{great_guy} )
\\
	&\mathrm{sim}( \texttt{united kingdom}, \texttt{Britain} ) = \mathrm{sim}( \texttt{united_kingdom}, \texttt{Britain} )
	\end{align}
$$

The weakness of this approach is rather obvious: it ignores compositionality altogether, and it is susceptible to data sparsity. 


### Selecting the distributional method
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
