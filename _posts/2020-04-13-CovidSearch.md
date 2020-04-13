---
layout: post
title: Covid19Search - A system for querying COVID-19 literature
---

Human have walked into an uncharted territory since the outbreak of the COVID-19 coronavirus. Much effort has been made to stop the crisis. For example, in machine learning community, some people have been seeking computational techniques for extracting insights from COVID-19 literature, such as the [COVID-19 Open Research Dataset Challenge (CORD-19)](https://www.kaggle.com/allen-institute-for-ai/CORD-19-research-challenge), which was also mentioned in [the news](https://www.wsj.com/articles/machine-learning-experts-delve-into-47-000-papers-on-coronavirus-family-11586338201?shareToken=st1ebd0a3a0e2e491b915d62fb96914cef&reflink=share_mobilewebshare).

Motivated by the community, I built a system for this purpose, I call it **Covid19Search** - a system for querying COVID-19 literature or finding insights from the literature. This post mainly explains the technical details behind the system. Below presents the overall process of the system in user customized query task and insights extraction task that are based on the [10 pre-defined questions](https://www.kaggle.com/allen-institute-for-ai/CORD-19-research-challenge/tasks) by CORD-19.


<object id="workflow" style="width: 100%" data="{{ site.baseurl }}/images/covid19search/covid19search.svg" type="image/svg+xml">
</object> <div style="text-align: center;font-size: 12px" id="figure1">Figure 1. The overall picture of Covid19Search</div>


For those just seeking a quick exploration for the system. Below are important places to go.

-  [Github Repository](https://github.com/wangcongcong123/covidsearch)
-  [Kaggle Notebook](https://kaggle.com/congcong123/search-for-finding-covid-19-related-insights)
-  [Live Demo Site](https://www.thinkingso.cf) 

If you find something interesting in [Figure 1](#figure1) and want to know more, I would  say the rest of this blog feeds you well.


### User Query Task

Basically, in this task, the core idea is to match a user's query with all documents (known as the corpus, academic articles in this case). The query is used to describe the user's information need such as the example in [Figure 1](#figure1): What drugs are helpful for COVID-19?  


The matching process is to caculate the similarities between the user's query and every single document in the corpus so that the most relevant documents can be returned to the user. That is at a high level how an information retrieval (IR) system (the daily-used Google for example) works. 


<object id="workflow" style="width: 100%" data="{{ site.baseurl }}/images/covid19search/covid19search-t1-1.svg" type="image/svg+xml">
</object> 

<div style="text-align: center;font-size: 12px" id="figure2">Figure 2. The relevance matching between a user query and a document</div>


Now, we need describe it in a mathematical way since human written like texts are unknown to machines. Machines are computers so they sheerly depend on numbers/digits, like viruses rely on living hosts otherwise they would die soon.

#### Vectorization

Hence, a conversion of texts to numbers is needed. The process of converting texts to numbers can be well explained by introducing **vectorization** (other names include weighting, embedding, etc.), as presented in the picture below.


<object id="workflow" data="{{ site.baseurl }}/images/covid19search/covid19search-t1-2.svg" style="width: 100%" type="image/svg+xml">
</object> <div style="text-align: center;font-size: 12px" id="figure3">Figure 3. The relevance matching between a user query and a document with vectorization</div>


Obviously, vectorization converts a varied-length sequence of texts to a fixed-length list of numbers (i.e., vectors) so relevance can be quantified actually by machines.

The converted vectors are important because they have to represent the texts very well. To say representing two texts that are assumed very relevant with vector 1 and 2, the objective here is to ensure each position of vector 1 should be close to that of vector 2 because the same position usually represents a shared latent feature of texts.

Given the importance, extensive research work has been made for this purpose. Some salient ones are given below.

- Count
- TF-IDF
- [BM25](https://en.wikipedia.org/wiki/Okapi_BM25)
- FastText

Figure 4 demonstrates the process of vectorization using the aforementioned two methods: **Count** and **TF-IDF**. The two methods are straightforward. Basically, before vectorization, a step known as **preprocessing**, is taken to build a vocabulary that contains all unique words in the corpus (see the left box of Figure 4).

The left box also shows a list of stop words used in preprocessing. The stop words are considered not to be informative for differentiating one document from another and thus they are removed from the list of vocabulary. Here some other steps in  preprocessing such as [stemming](https://en.wikipedia.org/wiki/Stemming), [lemmatization](https://medium.com/dair-ai/fundamentals-of-nlp-chapter-1-tokenization-lemmatization-stemming-and-sentence-segmentation-b362c5d07684), min-frequency filtering, etc., are omitted for demonstration convenience.

<object id="workflow" data="{{ site.baseurl }}/images/covid19search/covid19search-t1-3.svg" style="width: 100%" type="image/svg+xml">
</object> <div style="text-align: center;font-size: 12px" id="figure4">Figure 4. Count and TF-IDF vectorization</div>

The right box presents the process of count and TF-IDF vectorization on doc1. The count vectorizer is fed with the raw texts of each doc in corpus and generate a count vector with size the same as that of the vocabulary to represent the doc. 

The score or weight of each position in the count vector indicates the number of occurrences of the word that has the same position in the vocabulary list. For example, 2 at the third position refers to that "covid"(at the third position in the vocabulary list) occurs in doc1. 

The TF-IDF method works a bit differently from the Count method. It calculates a weight by multiplying term frequency (TF: actually the count mentioned in Count method) with inverse document frequency (IDF). See the right box of Figure 4 to know what they refer to.

Different from Count, IDF is introduced in this method to give more importance to the words that are not common across all documents. To put it in another way, in the example of Figure 4, the word "covid" occurs in every document in the corpus (it is ordinary and not outstanding across the corpus) and hence it is penalized to 0 (its IDF is 0 so its TF-IDF is 0). This is unlike  a very high score 2 given to "covid" in the Count method. In analogy, a pizza topped with sausage as well as jalapeño makes it stand out from all other pizzas topped with only sausage.



In TF-IDF, IDF is usually computed beforehand given the corpus is fixed and can be directly called to compute TF-IDF vector for a user query, which makes it more efficient of the retrieval process. 

For **BM25**, it is based on a probabilistic retrieval model with the introduction of extra parameters (k and b) as compared to TF-IDF. To know how the weight is exactly calculated in BM25, have a look at [this page](https://en.wikipedia.org/wiki/Okapi_BM25). 

To run any one of Count, TF-IDF, or BM25 for retrieving COVID-19 related papers with respect to a search query, have a look at [the script](https://github.com/wangcongcong123/covidsearch/blob/master/examples/full_text_run.py).

#### Word Embeddings

**FastText** is a word embedding method to learn linguistic information of texts through neural networks (NNs) based language model. 

Word embedding is a way to represent a word with fixed-length vectors of continuous real numbers. Due to this feature, it has been widely studied for representing text inputs for downstream NNs based language tasks such as classification, reading comprehension, translation and so on. This section only introduces its use in IR retrieval.

It  maps  a  word  in  a  vocabulary  to  a  latent  vector  space  where  words  with similar contexts are in proximity. Through word embedding, a word is converted to a vector that summarizes the word’s both syntactic and semantic information. That says the embedding process is similar to the vectorization process as mentioned.


Although a lot of methods have been proposed to learn embeddings in the literature, as a taxonomy in Figure 5, FastText embeds words by treating  each  word  as  being  composed  of  character  n-grams  instead of a word whole.  This allows it to not only learn rare words but also out-of-vocabulary words.

<object id="workflow" data="{{ site.baseurl }}/images/covid19search/covid19search-t1-4.svg" style="width: 100%;height: 50%" type="image/svg+xml">
</object> <div style="text-align: center;font-size: 12px" id="figure5">Figure 5. Context-independent embeddings vs context-dependent embeddings</div>

In addition, unlike context-based embeddings that requires re-construction of network architecture for generating contextualized word representations, FastText's pre-trained embeddings can be read directly and thus efficiently represent words without going through layers of NNs. As a starting point, Covid19Search now supports FastText for vectorizing words. 

To get a pre-trained FastText on the COVID-19 articles (CORD-19 dataset), have a look at the [script](https://github.com/wangcongcong123/covidsearch/blob/master/examples/embedding_run.py) to figure out easily. To represent a sequence of words, Covid19Search averages across FastText embeddings for individual words.

#### Relevance Measure

The vectorization process ensures a user query and every document in the corpus can be represented with vectors. With reference back to [Figure 3](#figure3), now the vectors are ready for relevance computation. Regarding relevance measure, imagining the vectors projected into a vector space, the purpose here is to calculate the distance between vector pairs. The closer distance indicates the closer relationship or to say the more relevant to each other. Below are a list of relevance measures that come to my mind immediately.

- Cosine Similarity
- [Hamming distance](https://en.wikipedia.org/wiki/Hamming_distance)
- [Correlation](https://en.wikipedia.org/wiki/Correlation_and_dependence)
- [Euclidean distance](https://en.wikipedia.org/wiki/Euclidean_distance)
- [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance)
- [Manhattan distance](https://en.wikipedia.org/wiki/Taxicab_geometry)
- [Jaccard index](https://en.wikipedia.org/wiki/Jaccard_index)

As seen, there are so many different ways to achieve this purpose, different methods have different use cases. For example, Levenshtein distance is known as edit distance that calculates the number of steps to edit a word to another word by insertion, deletion or substitution (such as 2 for coronavirus to coranavrius). Due to this feature, it is widely used in auto correcting misspellings. Covid19Search implemented cosine similarity for relevance measure, as calculated below.

<object id="workflow" data="{{ site.baseurl }}/images/covid19search/covid19search-t1-5.svg" style="width: 100%" type="image/svg+xml">
</object> <div style="text-align: center;font-size: 12px" id="figure6">Figure 6. Cosine similarity computation between vector A and B</div>

The calculation iterates over all documents in the corpus with the user query, ending up with a list of documents deemed relevant to the query. Documents are ranked by the relevance scores in descending order to return to users so that users can see the most relevant documents in the first place. This is how the user query task works in Covid19Search and generally how an IR system works in real world.


#### More on Covid19Search

There are two additional highlights of Covid19Search in the user query task
- An academic paper in this case usually contains three parts: title, abstract, and body. Covid19Search allows to assign different weights to each part in vectorization.
- Covid19Search supports for generating a final relevance score by combining the output of different vectorization methods, such as BM25 + FastText in the [script](https://github.com/wangcongcong123/covidsearch/blob/master/examples/ensemble_run.py).


### Insights Extraction Task

Insights extraction task matches [10 pre-defined questions](https://www.kaggle.com/allen-institute-for-ai/CORD-19-research-challenge/tasks) with the corpus for capturing relevant insights from the literature that are useful for answering the question. For example, one pre-defined question is: What do we know about COVID-19 risk factors? With this question, a retrieval system should target at finding important sentences from the literature that indicate useful information on COVID-19 risk factors. Motivated by this, Covid19Search leverages a recently-proposed sentence-level embedding method for this kind of semantic search. The method is known as [Sentence-BERT](https://github.com/UKPLab/sentence-transformers). Too long to read [the original paper](https://arxiv.org/pdf/1908.10084.pdf), here gives a simplified and brief describtion of how the technique is achieved in Covid19Search.

At a high level, Sentence-BERT is nothing but the vectorizer that converts a sentence/question to a fixed-length vector, working at the vectorization step as presented in [Figure 3](#figure3). Figure 7 presents the relevance matching between a pre-defined question and each sentence of each article of all in the corpus.  This says, an article is split into sentences and then feed the sentences one by one to Sentence-BERT that help encode the sentences to vectors. The process is the same applied to the pre-defined question so that the final relevance can be computed.

<object id="workflow" data="{{ site.baseurl }}/images/covid19search/covid19search-t2-1.svg" style="width: 100%" type="image/svg+xml">
</object> <div style="text-align: center;font-size: 12px" id="figure7">Figure 7. Vectorization using Sentence-BERT for matching a pre-defined question with sentences</div>

What makes Sentence-BERT outstanding is that it treats every sentence as a whole to generate a vector representation instead of individual words within the sentence as described in the user query task (bag-of-words). Hence, the representation is able to reveal the sentence's meaning as a whole to some extent. Indicated by its name, it is evident that Sentence-BERT leverages the state-of-the-art transformer based BERT model (now it supports other transformer based models such as [RoBERTa](https://arxiv.org/abs/1907.11692), [DistilBERT](https://arxiv.org/abs/1910.01108), [ALBERT](https://arxiv.org/abs/1909.11942)) for such general sentence embedding purpose (it actually uses siamese BERT-Networks pre-trained on inference datasets).

Delving into the depth of BERT or Sentence-BERT is beyond the scope of this blog. However, for whoever is interested in knowing about BERT, the two papers ([Transformer](https://arxiv.org/pdf/1706.03762.pdf) and [BERT](https://arxiv.org/abs/1810.04805)) are recommended to read first. If not satisfied with the understanding of BERT after reading the two papers, the two tutorials [the annotated transformer](https://nlp.seas.harvard.edu/2018/04/03/attention.html) and [The Illustrated BERT](http://jalammar.github.io/illustrated-bert/) are recommended. 

Despite the good feature of Sentence-BERT, it is not used in the user query task because it takes too much time to compute the relevance between an arbitrary user defined query (this needs to be re-computed for different queries every time) and all sentences (much more than the number of documents) in the corpus. Since the insights extraction task has a fixed query space (i.e, the 10 kaggle questions), the process of relevance matching can be computed beforehand and saved to [a pre-matched file](https://github.com/wangcongcong123/covidsearch/tree/master/models_save/sentencesearch). In real-time retrieval, the file can be directly loaded for efficient retrieval, see the [demo](https://www.thinkingso.cf/).

Last, thanks to [JiaZheng](https://jiazhengli.com/) who kindly provided the computation resources for semantically pre-matching sentences using [sentences-transformers](https://github.com/UKPLab/sentence-transformers). 

If you have any thoughts you'd like to share with me about the blog, send me an email via [wangcongcongcc@gmail.com](mailto:wangcongcongcc@gmail.com) or you are welcome to talk with me on [my Twitter](https://twitter.com/WangcongcongCC).


<!-- <div class="form-group">
  <label for="comment">Comment:</label>
  <textarea class="form-control" rows="5" id="comment"></textarea>
</div>

<button type="button" class="btn btn-primary btn-sm">Submit</button> -->