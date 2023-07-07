---
layout: post
title: Similarity-based Generalization in N-gram models
---

If you're unfamiliar with N-gram models, I suggest you to read my [introduction to N-gram language models](../ngram_intro) before continuing.

Using a finite set of words, we can produce infinite number of phrases and sentences. As N increases, the number of possible N-grams also increases exponentially, but the amount of N-grams we can observe from a finite text corpus is always linear to the size of the corpus. This is the data sparsity problem, which is the most fundamental problem in N-gram language modeling. In the previous post, I've mentioned many approaches that try to handle this issue, and *Similarity-based Generalization* is in my opinion the most important one of them. This idea which was originally proposed for bigram models, comes from the intuition that, when two words are semantically similar, the distribution of the words that proceed them are also similar. 

Here's a simple example: if "orange", "lemon", and "grapefruit" are considered similar and "orange juice", "lemon juice" have been observed in the training data, but "grapefruit juice" hasn't, we can estimate the probability of "juice" appearing after "grapefruit" using the probabilities of "juice" appearing after "orange" and "lemon". 

A relevant question is how do we determine which words are considered similar. The answer is distributional similarity. To get an idea about what distributional similarity is, I suggest reading my [introduction to Distributional Semantics](../distributional_semantics) first.


## The case for bigrams
The similarity based bigram model (Dagan et al., 1998) contains 3 essential components: 

1. $$S(w_1)$$: a set of words that are similar to the context word $$w_1$$; 
2. $$W(w_1^ \prime, w_1)$$: the rescaled similarity score between $$w_1$$ and  $$w_1^ \prime$$, where $$w_1^ \prime$$ is a member of $$S(w_1)$$;
3. $$P_{MLE}( w_2 \mid w_1 )$$ and $$P_{MLE}( w_2 )$$: the Maximum Likelihood Estimation (MLE) of the probability of $$w_2$$ given $$w_1$$, as well as the unigram probability of $$w_2$$

The final probability estimation ($$P_{r}$$) results from the interpolation between a unigram model $$P_{MLE}(w_2)$$ and a similarity based model $$P_{SIM}(w_2 \mid w_1)$$, where the interpolation is controlled by a hyperparameter $$\gamma$$ . $$P_{SIM}(w_2 \mid w_1)$$ is obtained by doing a weighted average over the MLE probabilities of all similar bigrams, where the weights are obtained by rescaling and normalizing the similarity scores between each word in $$S(w_1)$$ and $$w_1$$:
$$
P_{SIM}(w_2 \mid w_1) = \sum_{w_1^ \prime \in S(w_1)}P_{MLE}(w_2 \mid w_1^ \prime)\dfrac{W(w_1, w_1^ \prime)}{\sum_{w_1^ \prime \in S(w_1)}W(w_1, w_1^ \prime)}
$$

$$
P_{r}(w_2 \mid w_1) = \gamma P_{MLE}(w_2) + (1 - \gamma)P_{SIM}(w_2 \mid w_1)
$$

The choice of similarity measure and rescaling function is another important design decision. The authors used negative exponential KL divergence in their language modeling experiments:
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

In their experiments, this model achieved 20% percent improvement in perplexity over Katz backoff model (Katz, 1987). 

## Generalizing to higher order N-grams

The interpolative nature of the bigram model can easily be extended to arbitrary order N-grams in a recursive manner: the final probability estimation ($$P_r$$) of the N-gram model results from the interpolation between $$P_{SIM}$$ of the current order and $$P_r$$ of the (N-1)th order . At the lowest order (which is a unigram model), $$P_r$$ is simply equal to $$P_{MLE}$$:

$$P_{r}(w_i \mid w_{i-n+1 : i-1}) = \gamma P_r(w_i \mid w_{i-n+2 : i-1}) + (1-\gamma)P_{SIM}(w_i \mid w_{i-n+1 : i-1})$$

$$P_{r}(w_i) = P_{MLE}(w_i)$$

The more challenging question is the calculation of $P_{SIM}$. As a first step, we could just simply extend the equation for bigrams to N-grams:
$$
\begin{gather}
	P_{SIM}(w_{i} \mid w_{i-n+1 : i-1}) 
	= \sum_{w_{i-n+1 : i-1}^ \prime \in S(w_{i-n+1 : i-1})}P_{MLE}(w_{i} \mid w_{i-n+1 : i-1}^ \prime)
	\dfrac {W(w_{i-n+1 : i-1}^ \prime, w_{i-n+1 : i-1})}
	{\sum_{w_{i-n+1 : i-1}^ \prime \in S(w_{i-n+1 : i-1})}W(w_{i-n+1 : i-1}^ \prime, w_{i-n+1 : i-1})}
\end{gather}
$$
But functions such as $$S(w_{i-n+1 : i-1})$$ and $$W(w_{i-n+1 : i-1}^ \prime, w_{i-n+1 : i-1})$$ requires us to clearly define what it means for two N-grams to be similar.  It's obvious that at this point we're dipping our toes into the deep waters of compositionality. Here I propose 3 candidates:

### 1. potential definitions of N-gram similarity
#### Mean of Constituents (MoC)
As the name suggests, the similarity of two N-grams is simply defined as the average distributional similarity of each pair of words in both N-grams. For example: 
$$
\mathrm{sim}( \texttt{good man}, \texttt{great guy} ) = 
\dfrac{
	\mathrm{sim}( \texttt{good}, \texttt{great}) + \mathrm{sim}( \texttt{man}, \texttt{guy})
}
{2}
$$
This works well on certain examples, however it fails when N-grams are not compositional, for example it fails to capture the similarity between $$\texttt{best man}$$ and $$\texttt{groom's person}$$, because $$\texttt{best}$$ and $$\texttt{groom's}$$ aren't really synonymous.

#### Additive Composition and Multiplicative Composition
Elementwise vector addition/multiplication is one of the simplest yet effective methods in Compositional Distributional Semantics. It suggests that the distributional vector representation of a phrase can be derived by simply adding/multiplying the vectors of its constituents together. Using this approach, we can generate vector representations for each N-gram, and simply treat them as word vectors. Here's the same example:
$$
\mathrm{sim}( \texttt{good man}, \texttt{great guy} ) = 
\mathrm{sim}( \texttt{good} + \texttt{man}, \texttt{great} + \texttt{guy} )
$$
or 
$$
\mathrm{sim}( \texttt{good man}, \texttt{great guy} ) = 
\mathrm{sim}( \texttt{good} \odot \texttt{man}, \texttt{great} \odot \texttt{guy} )
$$
The biggest problem with this approach, is that it completely ignores word order. This means, $$\texttt{car company}$$ and $$\texttt{company car}$$ will have exactly the same representation. Moreover, similar to Mean of Constituent, Additive Composition also fails at non-compositional examples.

#### N-grams as Words
This can be seen as complete opposite of the previous two methods. It basically treats all N-grams as if they were words, and uses their neighboring words to construct their distributional representations, while ignoring all information from their constituents. This can be seen as the application of distributional semantics to N-grams. In theory, using this approach N-grams can be directly compared to words, or N-grams of any arbitrary order.
$$
\begin{align}
	&\mathrm{sim}( \texttt{good man}, \texttt{great guy} ) = \mathrm{sim}( \texttt{good\_man}, \texttt{great\_guy} )
\\
	&\mathrm{sim}( \texttt{united kingdom}, \texttt{Britain} ) = \mathrm{sim}( \texttt{united\_kingdom}, \texttt{Britain} )
	\end{align}
$$
The shortcoming of this approach is rather obvious: it ignores compositionality, and it suffers from N-gram sparsity. 

to be continued...
