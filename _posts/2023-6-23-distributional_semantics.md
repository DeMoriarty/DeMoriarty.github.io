---
layout: post
title: Introduction to Distributional Semantics
---

What makes *car* and *automobile*, *rage* and *anger* synonymous? A simple answer would be: they can be used interchangeably in many situations/contexts. This observation is where the intuition behind the **Distributional Hypothesis** came from: **words that occur in the same contexts tend to have similar meanings**. 

Distributional Hypothesis allows us to model some aspects of lexical semantics using statistics. In order to understand how it works, you only need some basic linear algebra:

Imagine words as points that residing in a $$d$$-dimensional Euclidean space. The coordinates of each word in this space, is described by a sequence of $$d$$ real numbers, which is called a *vector*, let's denote the vector of the word $$\textrm{w}$$ as $$\vec{v}_{\textrm{w}}$$ . Words that are semantically similar are also close to each other in this space, where "closeness" is determined by some sort of distance metric. The simplest type of distance is the Euclidean distance, which is basically the length of the shortest path between two points. However, in practice, Cosine Distance (or Cosine Similarity) - the cosine of the angle between two vectors - tends to yield the best results.

Let's address the important question of how we determine the coordinates of words. The answer is actually quite simple: let's assume $$d$$ is equal to $$\mid V \mid$$, which represents the size of the vocabulary containing all unique words in the text corpus. Each dimension of the vector $$\vec{v}_{\textrm{w}}$$ corresponds to a word in the vocabulary. To begin, we go through a large text corpus and count how many times $$\textrm{w}$$ cooccurs with other words in the same context. These cooccurrence counts are then recorded in the dimensions of $$\vec{v}_{\textrm{w}}$$ that correspond to the cooccurring words. For example, if "cat" cooccurs with "food" seven times, and "cat" is the 1385th word in our vocabulary, we assign the value seven to the 1385th dimension of $$\vec{v}_{\textrm{food}}$$. After repeating this process sufficiently, we obtain the conditional frequency distribution for all words in the vocabulary, given their cooccurrence with $$\textrm{w}$$. 

That's it! This represents the most basic form of a distributional representation. The simplest use case for distributional representations is finding words that are semantically similar to a given word. To achieve this, we compute the similarity between the vector of the given word and the vectors of every other word in the vocabulary. We then retrieve the words with the highest similarity scores.

However, if we attempt to apply this method to the count-based (or frequency-based) representations we obtained earlier, it may not yield satisfactory results. This is due to Zipf's law, which observes that in any language, a small set of words occurs extremely frequently, while the majority of words appear sparsely. For instance, in an English text corpus, "the" may occur millions or billions of times, whereas words like "defibrillator" occur only a few times. As a result, raw frequency distributions tend to be highly imbalanced, with a few dimensions having significantly larger values than the rest. These dominant dimensions tend to "overwhelm" the calculation of similarity scores. To address this, we need to balance out or penalize those highly magnitude but less meaningful dimensions through a process called **rescaling**.

One simple yet effective rescaling method involves using logarithms. Logarithms preserve the sorted order or rank of vector dimensions while compressing all the values into a smaller range. Essentially, it implies that frequent context words are only slightly more important than less frequent ones. It's worth noting that since frequencies can be zero, and $$\log(0)$$ is undefined, we have to add a small positive value to all cooccurrence counts before computing the logarithm.

Another straightforward approach is converting cooccurrence counts into binary values: 0 if there is no cooccurrence, and 1 if there is. At this point, the binary word vectors are equivalent to sets, and we can use Jaccard similarity to measure the similarity between them.

The most popular and best performing rescaling method is Pointwise Mutual Information (PMI):

$$
\textrm{PMI}(w_1, w_2) = 
\log\dfrac{P(w_1, w_2)}{ P(w_1)P(w_2) }
$$

The concept behind Pointwise Mutual Information (PMI) is actually quite intuitive. Not all cooccurrences between words carry meaningful information; some are merely coincidental. Certain words may cooccur not due to any inherent relationship, but simply because one or both of the words are commonly used and can accidentally appear alongside unrelated words. The cooccurrence of two words should be deemed "meaningful" only when it is not by chance. PMI compares the actual probability of two words cooccurring to what that probability would be if the words were independent.

$$
\begin{align}
    & P(w_1, w_2): \textrm{the observed joint probability of $w_1$ and $w_2$} \\
    & P(w_1)P(w_2): \textrm{the joint probability of $w_1$ and $w_2$ if they were independent}
\end{align}
$$

If $$w_1$$ and $$w_2$$ were completely independent, $$P(w_1, w_2)$$ would equal $$P(w_1)P(w_2)$$, resulting in a ratio of 1 and a logarithm of 0. In other words, the PMI of irrelevant words would be 0. When $$P(w_1, w_2) > P(w_1)P(w_2)$$, the PMI is positive, indicating a positive dependency between $$w_1$$ and $$w_2$$. A negative PMI implies a negative dependency, indicating that the two words co-occur less frequently than if they were independent. Negative PMI is more challenging to interpret and utilize compared to positive PMI, so in practice, we typically focus on Positive PMI (PPMI):

$$
\textrm{PPMI}(w_1, w_2) = \max(\textrm{PMI}(w_1, w_2), 0)
$$

Another crucial aspect of distributional word representations is determining which words should be considered as cooccurring, or defining what constitutes the "context." Some approaches define context loosely as "words appearing before or after the target word within a certain proximity," while others have stricter definitions, such as "words that have specific syntactic dependencies with the target word." The challenge with the latter approach is its reliance on external dependency parsers.

Lastly, let's discuss compression. If the dimensionality of each word vector equals the number of words in the vocabulary ($$\mid V \mid$$), and each word in the vocabulary has its own word vector, stacking all the vectors would result in a matrix of size $$\mid V \mid \times \mid V \mid$$, which can be exceedingly large. To improve memory and computational efficiency, we can compress this matrix through dimensionality reduction. There are generally two approaches to reducing vector dimensions. The first involves employing matrix decomposition methods like Singular Value Decomposition (SVD) or Principle Component Analysis (PCA). The second approach is "learning" decomposed matrices directly using Stochastic Gradient Descent (SGD).
