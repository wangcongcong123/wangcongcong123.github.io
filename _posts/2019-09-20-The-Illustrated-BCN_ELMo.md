---
layout: post
title: The Illustrated BCN+ELMo and its Application in Priority Classification 
---

This blog first details the internals of BCN+ELMo model for text classification and then apply it in a real-world classification problem. 

## Table of Contents
- Internals of BCN+ELMo for text classification
- An illustrated example to explain it
- Its application in prioriy classification for crisis-related tweets
- Conclusion


#### 1. Internals of BCN+ELMo for text classification

As the example given in [AllenNLP](https://allennlp.org/), the model is used for [SST-5](https://nlp.stanford.edu/sentiment/index.html) sentiment analysis. Here, I will talk about how to adapt it to another domain - priority classification for crisis relevant tweets. The dataset can be downloaded through this [link](/files/train_dataset_gt.json). Before going to apply the model into such a real-world use case. It is necessary first to look at the internals of how the model works. The picture below shows the architecture of the BCN integrated with ELMo model.

![_config.yml]({{ site.baseurl }}/images/bcn_elmo_architecture.png)


A good way to know well the internals of the architecture is to apply breakpoint debugging on the [implementation](https://github.com/allenai/allennlp/blob/master/allennlp/models/biattentive_classification_network.py) by AllenNLP with PyCharm IDE. To avoid hacking code, next I will give an example to explain it and hopefully help understand the model.

#### 2. An illustrated example to explain it

Pending to finish (include a special exmaple to explain the graph, especially the dimention transformation annotated)...

#### 3. Its application in prioriy classification for crisis-related tweets

Pending to finish (include source code implemented to adapt the model to a new domain)...

#### 4. Conclusion

Although the BCN_ELMo is ranked the 3rd in the state-of-the-art(SOTA) [leadboard](https://paperswithcode.com/sota/sentiment-analysis-on-sst-5-fine-grained) for SST-5 task, as of writing this blog, knowing its internals and application is a good starting point for subsequent research in NLP, espcially for language classification problems. In order to know the mathematics of the model. Two papers are worth exploring: [Peters, Matthew E., et al. 2018](https://arxiv.org/pdf/1802.05365.pdf) and [McCann, Bryan, et al. 2017](https://arxiv.org/pdf/1708.00107.pdf). Additionally, this [tutorial](http://www.realworldnlpbook.com/blog/improving-sentiment-analyzer-using-elmo.html) also helps a lot understand what is behind the ELMo and how it works.

If you have any doubts or any my mistakes you found in the post, send me an email via [wangcongcongcc@gmail.com](mailto:wangcongcongcc@gmail.com) or you are welcome to talk with me about any NLP relevant questions through [my Twitter account](https://twitter.com/WangcongcongCC).

 