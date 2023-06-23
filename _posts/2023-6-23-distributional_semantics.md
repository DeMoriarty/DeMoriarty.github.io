---
layout: post
title: Introduction on Distributional Semantics
---

What makes *car* and *automobile*, *rage* and *anger* synonymous? A simple answer would be: they can be used interchangeably in many situations/contexts. This observation is where the intuition behind the **Distributional Hypothesis** came from: **words that occur in the same contexts tend to have similar meanings**. 

Distributional Hypothesis allows us to model some aspects of lexical semantics using statistics. In order to understand how it works, you only need some basic linear algebra:

Imagine words as points that "live" in a $$d$$-dimensional Euclidean space (like the 3-dimensional space that we live in, but with much higher dimensionality). The coordinates of each word in this space, is described by a sequence of $$d$$ real numbers, which is called a *vector*, let's denote the vector of the word $$\textrm{w}$$ as $$\vec{v}_{\textrm{w}}$$ . Words that are semantically closer, are also close to each other in this space, where "closeness" is determined by some sort of distance metric. The simplest type of distance is the Euclidean distance, which is basically the length of the shortest path between two points. However, in practice, Cosine Distance (or Cosine Similarity), the cosine of the angle between two vectors, tends to perform the best.

Now the most important question is, how do we find out the coordinates of words? It turns out that the answer is actually very simple: assume $$d$$ equals to $$| V |$$, the size of the vocabulary, which is the set of all unique words in the text corpus. Each dimension of $$\vec{v}_{\textrm{w}}$$ corresponds to one word in the vocabulary. First, we need to go through a large text corpus, and count how many times $$\textrm{w}$$ cooccurs with other words in the same context, and write down the cooccurrence counts in the dimensions of $$\vec{v}_{\textrm{w}}$$ that correspond to the cooccurring words. For example, if "cat" happens to cooccur with "food" 7 times, and "cat" is the 1385th word in our vocabulary, then we put 7 to the 1385th dimension of $$\vec{v}_{\textrm{food}}$$.
After doing this enough times, what we get is the conditional frequency distribution of all the words in the vocabulary given they cooccur with $$\textrm{w}$$. 

That's it! this is the most basic form of a distributional representation! The simplest use case of distributional representations, is finding words that are semantically similar to a given word. To do so, we can just compute the similarity between the vector of the given word with vectors of every other word in the vocabulary, and retrieve the words with largest similarity scores. 

But if you try this on the count based (or frequency based) representations we got previously, it may not work very well. 

The reason is Zipf's law, it comes from the observation that in any language, there is a small set of words that occur extremely frequently, while the vast majority of the words appear very sparsely. For example, in any English text corpus, *the* may occur millions or billions of times, while something like *defibrillator* occur only a handful of times. Because of this, raw frequency distributions tend to be highly imbalanced, where few dimensions have extremely large values than the rest, and those few dimensions tend to "dominate" the result of similarity score calculation. This is why we need a **rescaling** step. 

A very simple yet effective rescaling method, is logarithm. Logarithm preserves the sorted order of the vector dimensions, while squishing all the values into a much smaller range. It's basically a way of saying: frequent context words are only slightly more important than the less frequent ones. Because frequencies can be 0, and $$\log(0)$$ is undefined, we also need to add a small positive value to all the co-occurrence counts before computing logarithm. 

Another simple method, is converting cooccurrence counts into binary values: 0 if no cooccurrence, 1 otherwise. At this point, the binary word vectors are equivalent to sets, and we can use Jaccard similarity to measure the similarity of two sets.

The most popular and best performing rescaling method is Pointwise Mutual Information (PMI):

$$
\textrm{PMI}(w_1, w_2) = 
\log\dfrac{P(w_1, w_2)}{ P(w_1)P(w_2) }
$$

Don't be intimidated by the math, the idea behind PMI is actually very intuitive: Not all cooccurrences are meaningful, some of them are just incidental. Some words cooccur not because they have any sort of dependency relation, but simply because one or both of the words are universally frequent, and may accidentally appear next to irrelevant words. The cooccurrence of two words should only be considered "meaningful" when the cooccurrence is not accidental. PMI compares the probability of two words cooccurring together with what this probability would be if those words were independent.

$$
\begin{align}
	& P(w_1, w_2)  : \textrm{ what is the joint probability of $w_1$ and $w_2$ based on our observation} \\
	& P(w_1) P(w_2) : \textrm{ what would be the joint probability of $w_1$ and $w_2$ if they were independent}
\end{align}
$$

If $$w_1$$ and $$w_2$$ were completely independent, $$P(w_1, w_2)$$ should be equal to $$P(w_1) P(w_2)$$, so the ratio between them would be 1, and $$log(1) = 0$$. In short, PMI of irrelevant words will be 0. When $$P(w_1, w_2) > P(w_1) P(w_2)$$, the PMI will be positive, we say $$w_1$$ and $$w_2$$ are positively dependent. A negative PMI would imply negative dependency, which translates to "two words co-occurring less frequently than they would if they were independent". Negative PMI is harder to interpret and make use of compared to positive PMI, so in practice we usually only care about PPMI (Positive PMI):

$$
\textrm{PPMI}(w_1, w_2) = \max( \textrm{PMI} (w_1, w_2), 0)
$$

Another important detail in distributional word representations is determining which words should be considered as cooccurring, in other words defining what "context" is. Some methods define context loosely as "words that appear before or after the target word within certain proximity", while some other methods have much stricter definitions, such as: "words that have certain syntactic dependency with the target word". The problem with the second type, is that they rely on external dependency parsers. 

Last thing I will talk about is compression. If the dimensionality of each word vector is equal to the number of words in the vocabulary $$\mid V \mid$$ , and each word in the vocabulary also has its own word vector, when we stack all vectors on top of each other, we would get a $$|V|\times|V|$$ matrix, which can be enormous. For memory and computational efficiency, we can compress this matrix by performing dimensionality reduction. There are broadly two ways of reducing vector dimensions. First one is using some matrix decomposition method like Singular Value Decomposition (SVD) or Principle Component Analysis (PCA). Second way is "learning" decomposed matrices directly with Stochastic Gradient Descent (SGD). 
