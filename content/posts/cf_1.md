---
title: "Large-scale image board archival"
date: 2019-09-13T09:30:47-04:00
tags: ["scraping"]
draft: false
author: "simon987"
---

# *chan Crawler Overview
Image boards are volatile by design and require special considerations when scraping 
(especially when dealing with dozens of them at the same time!). 
This is an overview of my implementation of a infrastructure that reliably collects and processes
~150 GiB/day of data from over 27 image boards in real time.



{{< figure src="/cf/grafana.png" title="">}}

## Core

The core of the crawler is very straightforward: most of the
work depends entirely on the website, as some boards have simple APIs with JSON endpoints
and others require more complex HTML parsing.

{{<highlight python >}}
scanner = ChanScanner(helper=CHANS["4chan"]) 
publish_queue = queue.Queue()

for item, board in scanner.all_posts():
    # Publishing & post-processing is done asynchronously on separate threads
	publish_queue.put((post, board))
{{</highlight>}}

## Deduplication

To avoid publishing the same item twice, the application keeps track of what items were visited in its **state**. 
Items that have the same `last_modified`, `reply_count` or `timestamp` value as the state doesn't need to be visited again.

This deduplication step greatly reduces the amount of HTTP requests necessary to stay up to date, and more importantly,
it enables the crawler to quickly resume where it left off in the case of a fault.

{{<highlight python >}}

# The state is saved synchronously to a SQLite database
state = ChanState()

def once(func):
    """Ensures that a function is called at most once per item"""
    def wrapper(item):
        if not state.has_visited(item):
            func(item)
            state.mark_visited(item)

    return wrapper

@once
def publish(item):
	# Publish item to RabbitMQ...


{{</highlight>}}


## Rate-limiting

A similar approach is used to rate-limit HTTP requests. The `rate_limit` decorator is
applied to the `self._get` method, which enables me to set a different `reqs_per_second` value
for each website. Inactive boards only require a request every 10 minutes, while larger ones
require at least 1 request per second to stay up to date.

{{<highlight python >}}
class Web:
    def __init__(self, reqs_per_second):
        self.session = requests.Session()

        @rate_limit(reqs_per_second)
        def _get(url, **kwargs):
            return self.session.get(url, **kwargs)

        self._get = _get
{{</highlight>}}

## Post-Processing

Before storing the collected data, there is a post-processing step where I parse the post body
to find images and URLs pointing to other domains. This information might be useful in the
future, so we might as well do it while the data is in memory.

All images are downloaded and their hashes are calculated for easy image comparison.
Checksums can be used for exact matching and image hashes (`ahash`, `dhash`, `phash` and `whash`)
can be used for *fuzzy* image matching (See [Looks Like It - Algorithms for comparing pictures](https://www.hackerfactor.com/blog/index.php?/archives/432-Looks-Like-It.html)). 

This kind of information can be used by data scientists to track the spread of an image on the internet, 
even if the image was slightly modified or resized along the way.

{{<highlight json >}}
{
  "_v": 1.5,
  "_id": 100002938,
  "_chan": 25,
  "_board": "a",
  "_urls": [
    "https://chanon.ro/a/src/156675779970.jpg"
  ],
  "_img": [
    {
      "url": 0,
      "size": 109261,
      "width": 601,
      "height": 1024,
      "crc32": "e4b7245e",
      "md5": "a46897b44d42d955fac6988f548b1b2f",
      "sha1": "5d6151241dfb5c65cb3f736f22b4cda978cc1cd0",
      "ahash": "A/w/A/A/A/G/e/A/E/A/A/A/",
      "dhash": "cY+YqWOaGVDaOdyWGWOUKSOS",
      "phash": "jVVSItap1dlSX0zZUmE2ZqZn",
      "whash": "Dw8PDw8PDw8="
    }
  ],
  "id": 2938,
  "html": "<table>...</table>",
  "time": 1566782999,
  "type": "post"
}
{{</highlight >}}

## Archival

{{< figure src="/cf/pg.png" title="">}}

The storage itself is handled by the **feed_archiver** process, which is
completely independent of the \*chan scraper (in fact, the exact same application is
used to store [reddit_feed](https://github.com/simon987/reddit_feed) data). 

The application will automatically create new tables as needed, and will store the items to
PostgreSQL.

{{< figure src="/cf/diagram.png" title="">}}

Data is saved in a `JSONB` column for easy access. 
Retrieving all posts made this year is possible with plain SQL:

{{<highlight sql >}}
SELECT *
FROM chan_7chan_post
WHERE (data->>'time')::INT >= '2019-01-01'::abstime::INT
{{</highlight >}}


# Real-time visualization

Because we're working with a message queue, it's trivial to attach other components to
the pipeline. As an experiment, I made a [WebSocket adapter](https://github.com/simon987/ws_feed_adapter)
and an [application](https://github.com/simon987/feed_viz) that displays the images as they are ingested in
your web browser.

{{< figure src="/cf/_chan.jpg" title="">}}

The live feed can be accessed [here](https://feed.the-eye.eu).

