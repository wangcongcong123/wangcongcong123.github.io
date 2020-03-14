---
layout: post
title: How to listen to arXiv based on keywords and post identified updates to Twitter?
---

This post guides you to write a python script that is used to monitor the open-access repository of electronic preprints ([arXiv](https://arxiv.org/)) for automatic post on Twitter. The workflow is illustrated as follows.




Basically, for every seconds of REQUEST_INTERVAL (3600 by default), a request through the [arXiv API](https://arxiv.org/help/api) is sent to get a update in relation to the topic based you the ARXIV_QUERY (bert+nlp by default. You can adapt it to the topic you prefer) you provided. If the update is deemed the latest, it will be forwarded and posted to Twitter autmatically via [the Twitter API](https://developer.twitter.com/en/docs).



#### Requirements

Twitter developer account.



``````python
import tweepy
import atoma
import time
from datetime import datetime
``````

Please go to [the code repository](https://github.com/wangcongcong123/Feeder-bot), where code is provided.

If you have any doubts or any my mistakes you found in the blog, send me an email via [wangcongcongcc@gmail.com](mailto:wangcongcongcc@gmail.com) or you are welcome to talk with me about any NLP relevant questions through [my Twitter account](https://twitter.com/WangcongcongCC).

<!-- <div class="form-group">
  <label for="comment">Comment:</label>
  <textarea class="form-control" rows="5" id="comment"></textarea>
</div>

<button type="button" class="btn btn-primary btn-sm">Submit</button> -->