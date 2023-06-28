---
layout: post
title: A Developing Framework for Natural Language Systems
---

In order to comprehend the human mind and create interpretable and reliable AI systems, I firmly believe in the necessity of a practical theory of natural language. While classical linguistic theories hold some truth and offer valuable insights into language's nature, they have not achieved significant success in the field of NLP. This can be primarily attributed to their neglect of a crucial aspect: language acquisition. The primary objective of this project is to develop a comprehensive framework for acquiring, understanding, and generating natural language, with particular emphasis on efficient learning and generalization from limited observations. By pursuing this goal, we aim to enhance our understanding of both machine and human intelligence, ultimately leading to the development of interpretable and robust AI systems.

## Guidelines and Principles
1. everything should be as general and as minimalistic as possible.
2. try not to make any exception or "special cases". Instead, make sure the framework is general enough to also have explanations for the exceptions.
3. acquisition of language should be automatic and unsupervised, it should not require any human effort after the initial design/coding phase. it should not rely on external tools such as parsers, pos taggers or WordNet.
4. never sacrifice interpretability for performance/accuracy. don't overly rely on black-box approaches.
5. there are shortcuts, and they work, don't be tempted by them. pick the hard path and endure the suffering.

## Main Assumptions and Hypotheses
1. Language is an interpretable system that can be modeled mechanistically.
2. In language, there are observable elements such as morphemes, words, phrases, sentences, and idioms. Additionally, there are latent elements that cannot be directly observed but can be interpreted, including case, tense, feature, semantic components and categories, syntactic rules and templates. I hypothesize that all observable and unobservable linguistic units (LINGU) are represented, learned, and participate in language processing in a similar manner. A similar hypothesis is also present in **construction grammar**: "it's constructions all the way down"
3. The underlying structure of any LINGU's representation involves a mapping between form and meaning. The **form** represents a pattern or regularity in the lower-level representation of text, or more specifically, it is the arrangement of lower-level LINGUs in a specific temporal or spatial order. This means, lower level LINGUs constitute higher level forms. Just like people are identified by their faces or names, LINGUs are identified by their form. 
4. According to Distributional Semantics, meaning is defined as the distribution of the contexts in which a LINGU may appear. In other words, it is an estimation about the other LINGUs that may appear next to a given LINGU. Meanings can vary in generality or specificity based on the number of forms associated with them. A meaning becomes more general when it is mapped from more forms. Hierarchical relationships, also known as "hypernymy-hyponymy relationship" in semantics, can exist between more specific and more general meanings.
5. The mapping between form and meaning is not strictly one-to-one. A form can correspond to multiple meanings, and a meaning can be associated with multiple forms. This characteristic allows for polysemy to exist.
6. The mapping between form and meaning is assumed to be arbitrary, meaning there is no simpler function than the mapping itself that can transform the form into meaning or vice versa. In other words, even if there is a possibility, we assume that the meaning cannot be predicted from the form. This means, the mapping between form and meaning must be memorized.
7. When encountering a novel form without an associated meaning or when the existing associated meanings are inappropriate in a given context, the intended meaning of the form is estimated using information from various sources. These sources may include:
	1. meanings of other memorized forms that are similar to the novel form
	2. meanings of the constituents of the novel form
	3. meanings of other memorized forms that can be constituted from the constituents of the novel form.
	4. context surrounding the novel form 
	This process of estimating meaning involves compositionality, which will be further discussed later posts.

To be continued...
