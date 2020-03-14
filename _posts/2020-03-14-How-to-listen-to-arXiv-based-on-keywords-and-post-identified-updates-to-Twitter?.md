---
layout: post
title: How to listen to arXiv based on keywords and post identified updates to Twitter?
---

This post guides you to write a python script that is able to monitor the open-access repository of electronic preprints ([arXiv](https://arxiv.org/)) for automatic post on [Twitter](https://twitter.com/home), named Feeder-bot. Its workflow is illustrated as follows.

<object id="workflow" data="{{ site.baseurl }}/images/feeder-bot-arxiv-twitter/feeder-bot-arxiv-twitter.svg" type="image/svg+xml">
</object>

Basically, Feeder-bot listens to a RSS feed provider (arXiv in this example) through its API ([arXiv API](https://arxiv.org/help/api) in this example) and forward and post any identified updates to the receiver (Twitter in this example) via its API ([Twitter API](https://developer.twitter.com/en/docs)).

For the code details, refer to [the repository](https://github.com/wangcongcong123/Feeder-bot). For how I built this tool, follow the rest of this blog.


#### Motivations

Below are the major motivations for me writing this tool.

- I recently rent a VPS from [Vultr](https://www.vultr.com/). I want to try something interesting on it. Hence, Feeder-bot become one of many.
- I am studying some open source projects these days such as [Transformers](https://github.com/huggingface/transformers), [AllenNLP](https://github.com/allenai/allennlp), [Flair](https://github.com/flairNLP/flair), etc. These are popular open source repositories in NLP. In addition, I spent some time learning Docker that is used to employ a project easily. Moviated, I want try something to practice the open source skills I learnt from there. Hence, I decided to write Feeder-bot.
- I usually look around on Twitter for knowing paper updates in relation to my research field, namely, NLP stuff. I does not use arXiv derivatives such as [arXiv-sanity](https://www.arxiv-sanity.com/) (that is actually a pretty nice work by [Andrej Karpathy](https://twitter.com/karpathy)) for this purpose due to the network effect and mobile friendliness of Twitter. Considering this, it would be great if I can post paper updates from arXiv on Twitter without going to arXiv. In this sense, Twitter acts as a repository of storing relevant papers like arXiv-sanity without the bother of switching between platforms. Hence, I proposed Feeder-bot that is a bot saving me from posting it manually.
- The aformentioned basically says I got some needs from the context where I am in. The needs first prompted me to search online checking if there is any relevant work that have done and well suits my needs. It turned out to be no ones there. Hence, it subsequently prompted me to build it from scratch.

For whoever has the similar needs as me, I hope this tool is useful for you. Although the tool's name is Feeder-bot, the actual focus of this blog is on the arXiv2Twitter bot that works as a starting example (As in <a href="#workflow">the figure</a> illustrating this example) for expanding Feeder-bot to support more RSS feeds or receivers in the future. This open source tool welcomes everyone to join in and contribute to the future work.

#### Prerequisite

Prior to coding, I knew I have to gain a general knowledge on two stuff. One is the Twitter API that enables a bot to post on Twitter and the other one is [arXiv API](https://arxiv.org/help/api) that offers interfaces to access to its content. The former is a bit troublesome to get. You have to go to the [Twitter Developer website](https://developer.twitter.com/) and make an application for keys before using its APIs. (Hint: if you have trouble filling in the application form, google "twitter developer account application example" that may help). 

Assume you have completed this step, now you get four pieces of information. They are API key, API secret key, Access token and Access token secret. Keep them well and you will use them later. The arXiv API is easily to get access without the application process. Just check its [API documentations](https://arxiv.org/help/api/user-manual), and easy to figure out how to use. 

<blockquote style="font-size: 12px" id="arxiv-request-url">
http://export.arxiv.org/api/query?search_query=all:NLP&&start=0&max_results=1&sortBy=lastUpdatedDate&sortOrder=descending
<p># This is an example of arXiv URL (HTTP-GET) for requesting the latest paper that containes the keyword "NLP"</p>
</blockquote>

After gaining a general knowledge of the APIs, I searched if there are any open source implementations upon the APIs and tweepy is a good choice for Twitter. To use it, install it with the command: `pip install tweepy`.


<pre><code class="language-python" style="background: #fff">
import tweepy

ARXIV_QUERY="NLP" # the keyword(s) to query the latest paper from arXiv 

# LATEST_PAPER_URL and LATEST_PAPER_TITLE refer to arXiv link and link of the lastest paper respectively which will be mentioned later.
post_content= 'new arxiv '+ ARXIV_QUERY +' related paper: '+ LATEST_PAPER_URL'\n'+LATEST_PAPER_TITLE+"." # We concatenate a content to be posted to Twitter

# Twitter verification
auth = tweepy.OAuthHandler(TWITTER_APP_KEY,TWITTER_APP_SECRET) # TWITTER_APP_KEY is the API key, TWITTER_APP_SECRET is the API secret key
auth.set_access_token(TWITTER_KEY, TWITTER_SECRET) # TWITTER_KEY is the Access token and TWITTER_SECRET is the Access token secret
api = tweepy.API(auth)
# post status
response=api.update_status(post_content,tweet_mode="extended")
</code></pre>

 For arXiv API, the only task to take is to parse the Atom fromatted content returned via the API from arXiv (I have never heard of Atom prior to reading arXiv API). The package named atoma is a good choice for this task (figure this out by simply googling "python atom parser"). Having had a look at the quick start of this package, it is enough to be used for this tool. To install it, enter the command: `pip install atoma`.

<pre><code class="language-python" style="background: #fff">
# this is a snippet adapted from the quick start of atoma: https://pypi.org/project/atoma/
import atoma, requests
# we request the latest NLP paper using the URL provided by arXiv API
response = requests.get('http://export.arxiv.org/api/query?search_query=all:NLP&&start=0&max_results=1&sortBy=lastUpdatedDate&sortOrder=descending') 
# Considering the needs of this tool, we only need three meta data of this paper: update time, title, url link
feed = atoma.parse_atom_bytes(response.content)
# once feed is succesfully obtained, debugging it to know how to get update time, title, url link
# After debugging, they are feed.updated, feed.entries[0].title.value (i.e., LATEST_PAPER_TITLE), feed.entries[0].id_ (i.e., LATEST_PAPER_URL) respectively.
print(feed.updated)
print(feed.entries[0].title.value)
print(feed.entries[0].id_)
</code></pre>

At this stage, it is easy to write a script that monitors the latest paper on ARXIV_QUERY through its API and posts the paper's link (LATEST_PAPER_URL) and title (LATEST_PAPER_TITLE) if it is found. To decide if new relevant paper is uploaded on arXiv, the bot requests arXiv every REQUEST_INTERVAL seconds. To make the bot a bit more interesting and flexible, some extra parameters are added as well. Below give examples of explaining these parameters.

#### Some Examples

1.**Set ARXIV_QUERY to be "bert+OR+nlp" indicating the user's interest fields include both BERT and NLP.**
<pre><code class="language-python" style="background: #fff">python arxiv_twitter.py -q "bert+OR+nlp"
</code></pre>

2.**Same as the above except that the update from arXiv is checked every 1800 seconds.**
<pre><code class="language-python" style="background: #fff">python arxiv_twitter.py -q "bert+OR+nlp -r 1800"
	</code></pre>


3.**Same as the above except that only the papers on ariXv with the update date no more than 5 days ago are considered to be forwarded and posted to Twitter.**
<pre><code class="language-python" style="background: #fff">python arxiv_twitter.py -q "bert+OR+nlp" -r 3600 -d 5
	</code></pre>

4.**The papers containing the keywords both coronovirous and convid are tracked and post content is prepended with "#virus,#covid2019,#coronovirous".**
<pre><code class="language-python" style="background: #fff">python arxiv_twitter.py -r 3600 -q "coronovirous+AND+convid" -d 10 -t "#virus,#covid2019,#coronovirous"
	</code></pre>

Where `python arxiv_twitter.py -h`

<pre><code class="plaintext" style="color: #000;background: #fff">
usage: arxiv_twitter.py [-h] [-r REQUEST_INTERVAL] [-q ARXIV_QUERY]
                        [-d DAYS_SINCE] [-t HASHTAGS_PREPEND]

optional arguments:
  -h, --help            show this help message and exit
  -r REQUEST_INTERVAL, --request_interval REQUEST_INTERVAL
                        The request_interval (Default: 3600) means sending
                        request to arxiv api every seconds of
                        request_interval.
  -q ARXIV_QUERY, --arxiv_query ARXIV_QUERY
                        The arxiv_query (default: 'nlp+OR+bert') is specified
                        for automatic retreival of latest papers from arxiv.
                        Space should be replaced with '+'. More refers to:
                        https://arxiv.org/help/api.
  -d DAYS_SINCE, --days_since DAYS_SINCE
                        The days_since (default: 10) defines the number of
                        past days since considered to be an update, e.g., 10
                        means that ony the papers on ariXv with the update
                        date no more than 10 days ago are considered to be
                        forwarded and posted to Twitter.
  -t HASHTAGS_PREPEND, --hashtags_prepend HASHTAGS_PREPEND
                        The list of hashtags (default:
                        '#NLP,#MachineLearning') you want to prepend the
                        tweet, seperated by ','. Namely, post_content = HASHTAGS_PREPEND+post_content
</code></pre>


In conclusion, Feeder-bot helps you listen to updates from arXiv and automatically forward them to Twitter if updates found. You just need to specify the some arugments such as arxiv_query, hashtags_prepend to run the bot !
Please go to [the code repository](https://github.com/wangcongcong123/Feeder-bot), where code is provided.

I had no any idea of how to implement it in the beginning. Later on, I researched and completed this small program quickly. What I learnt from this experience is that one can build up a good stuff efficiently as long as he/she has solid basics of programming/knowledge and that stuff is what he/she really needs. The solid basics help him/her fix bugs quickly and thus reduce the pain in the process. That is what he/she needs makes him/her passionate about it and thus a final release is possible. 
<!-- What motivated me to write this blog is not the bot itself, but the high-level consistent patterns of completing things likewise. -->

If you have any thoughts you'd like share with me about the blog, send me an email via [wangcongcongcc@gmail.com](mailto:wangcongcongcc@gmail.com) or you are welcome to talk with me on [my Twitter](https://twitter.com/WangcongcongCC).

<!-- <div class="form-group">
  <label for="comment">Comment:</label>
  <textarea class="form-control" rows="5" id="comment"></textarea>
</div>

<button type="button" class="btn btn-primary btn-sm">Submit</button> -->