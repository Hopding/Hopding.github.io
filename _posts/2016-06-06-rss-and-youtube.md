---
layout: post
title: RSS and Youtube
comments: true
---
I've recently discovered that RSS feeds are a very efficient way to subscribe to blogs
and websites with content that is updated frequently. I don't use Twitter of Facebook much,
so I can't rely on social media posts to notify me of new content. I will sometimes subscribe
to a site via email, but not all sites offer that option, and it can be distracting to have
that type of content filling my inbox. I prefer to reserve my email inbox for communication,
and related such activities - not alerts and reminders.

I also find [Pocket](https://getpocket.com) to be a useful service for organizing content and
saving articles that I find interesting or want to read later. Browser bookmarks are nice as well,
but my bookmarks list tends to grow cluttered with other things. I like using Pocket because it has a
clean, informative, organized layout that is easily perused and managed.

Conveniently, I recently came across [Feedhuddler](https://feedhuddler.com) when looking for a
cross-platform RSS reader. It hooks into Pocket, and sends new content from the RSS feeds that
you subscribe to directly to your Pocket List!

Whenever I watch a video on Youtube, I inevitably find myself clicking through the related videos
section, and wasting time watching silly, pointless videos - or nice, quality videos that I just don't
have time to be watching. I use Firefox as my main browser, so I was able to make use of the wonderful
[CleanTube](https://addons.mozilla.org/en-US/firefox/addon/cleantube/) add-on to prevent myself from
getting distracted by related videos.

I still wanted to be able to watch new videos from channels that I subscribe to, but I
didn't want to receive updates in my inbox, nor did I want to have to go to the channels' dashboards and
check for new videos myself. Besides, I would have to turn off CleanTube every time I did this, and that
meant risking getting distracted by related videos.

Fortunately, after a bit of googling, I was able to find a solution to my problem: Youtube RSS feeds! Youtube
doesn't do a good job making their existence known, but all channels do have RSS feeds. This meant that I could
subscribe to my favorite channels' feeds with Feedhuddler, and see new videos in my Pocket List - perfect! Now I
would be able to access all my favorite web content from one centralized location, thus minimizing wasted time and
distractions.

Youtube RSS feed URLs are somewhat difficult to access. I followed these instructions from one of [Daniel Miessler](https://danielmiessler.com/about/)'s
[blog posts](https://danielmiessler.com/blog/rss-feed-youtube-channel/)[^0] (bold notes are mine):

>
>    1. Go to the YouTube channel you want to track
>    2. View the page’s source code **(note: just right click and press "View Page Source")**
>    3. Look for the following text: channel-external-id **(note: `ctrl+f` "channel_id" works too)**
>    4. Get the value for that element (it’ll look something like UCBcRF18a7Qf58cCRy5xuWwQ
>    5. Replace that value into this URL:
>
>    `https://www.youtube.com/feeds/videos.xml?channel_id=UCBcRF18a7Qf58cCRy5xuWwQ`
>
>    Now you can paste that into any RSS reader and you’ll be able to track when new content is posted.

### Footnotes
[^0]: Perhaps there is another, easier way to do this that I'm simply not aware of? If you're aware of one, please do share.
