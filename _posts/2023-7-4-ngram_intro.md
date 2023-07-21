---
layout: post
title: Introduction to N-gram language models
---
Language modeling has been a pivotal area in Natural Language Processing (NLP). It forms the foundation for several applications like speech recognition, machine translation, spell correction, and more. One of the initial and simplest techniques in language modeling is the N-gram model. 

In its most rudimentary form, an N-gram is a contiguous sequence of 'n' items from a given sample of text or speech. When we use N-grams for language modeling, we aim to predict a word based on its preceding 'n-1' words. For instance, in a bigram model (where n=2), we predict a word based on its immediate predecessor. 

The fundamental assumption underpinning N-gram models is the Markov assumption, which states that the probability of a word depends only on the previous 'n-1' words. This allows us to make a manageable simplification of the complexity inherent in language. 

To understand N-gram models, we first need to answer the question: How do we calculate the probability of the word "states" following "united"? A straightforward solution is to count how many times "united states" appears in a text corpus and divide this by the total number of times "united" is found. This is tantamount to asking, "What percentage of the words that follow 'united' are 'states'?". This is known as the Maximum Likelihood Estimation (MLE) of conditional probability, which is represented as follows:

$$
P(\texttt{states} \mid \texttt{united} ) = \dfrac{
	C(\texttt{united states}) 
}{
	C(\texttt{united})
}
$$

This can be generalized to arbitrary order N-grams as such:

$$
P(\texttt{be} \mid \texttt{to be or not to} ) = \dfrac{
	C(\texttt{to be or not to be}) 
}{
	C(\texttt{to be or not to})
}
$$

More formally, to estimate the probability of the i'th word in a sequence, given the previous n-1 words has occurred, we use the following equation:

$$
P(w_i | w_{i-n+1:i-1}) = \dfrac{
	C( w_{i-n+1:i} ) 
}{ 
	C( w_{i-n+1:i-1})
}
$$

N-gram models are a good starting point, but they do have noticeable shortcomings. The main issue is data sparsity, which means for larger 'n' values, the likelihood of finding an unknown N-gram in the test data grows, leading to inaccurate predictions. Adding more training data can help to some extent, but the benefits get smaller each time. Plus, sometimes, we just don't have enough training data. Therefore, we need to incorporate some generalization techniques to improve the model. Another problem is that the text is treated merely as a linear sequence of words, totally ignoring the language's syntactic structure. We will discuss this second issue in more detail in future posts.

## Generalization techniques

*Additive Smoothing*: This technique, which includes methods like Laplace smoothing and add-k smoothing, adds a certain number to the counts of every possible N-gram. This makes sure that no N-gram has a zero probability, which can be crucial when dealing with unseen N-grams in the test data.

*Discounting*: The fundamental idea of discounting is to subtract a constant from each observed count to allocate that probability mass to unseen N-grams. Good-Turing and Absolute Discounting are common methods used. This reduces the impact of rare events and helps in better generalization.

*Backoff*: Backoff methods, like Katz Backoff, reduce the N-gram order if an N-gram has never been encountered. For instance, if we haven't seen a certain trigram, we might defer to a bigram or unigram model. This provides a balance between higher-order and lower-order models, aiding in managing data sparsity.

*Interpolation*: With interpolation, we combine probabilities from different N-gram orders. These are typically weighted by a lambda coefficient. This technique allows us to draw on context from various lengths, leading to more balanced predictions.

*Hybrid*: Hybrid approaches combine several other types of techniques in order to further improve the performance of the N-gram model. The most well-known example of hybrid methods is Kneser-Ney smoothing. This technique not only reduces the count of seen N-grams, like discounting, but also estimates the probability of unseen N-grams in a more intelligent way. It takes into account how many different contexts a word has appeared in, not just the number of times it has appeared. This method gives a better chance to words used in diverse contexts and helps make more accurate predictions with new data. A modified version of Kneser-Ney smoothing is considered one of the most effective methods in this field.

*Analogy*: Techniques like similarity-based generalization and class-based generalization fall under Analogical Generalization, which is less common in N-gram language modeling. The basic idea here is that analogies between similar words can help estimate unseen N-gram probabilities. An in depth introduction to the similarity-based method can be found [here](../similarity_based). The existing similarity-based methods are only applicable to bigram models. In the future posts, I will delve deeper into the existing similarity-based bigram model, and introduce an improved algorithm that can be applied to n-grams of any order.

I highly suggest [this article from Stanford](https://web.stanford.edu/~jurafsky/slp3/3.pdf). It provides thorough explanations about N-gram language models and covers some of the methods I've discussed.
