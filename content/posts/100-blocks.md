+++
title = "100 Blocks a Day, quantified"
author = ["Ketan Agrawal"]
date = 2020-09-23T01:33:00-04:00
lastmod = 2020-09-23T01:56:12-04:00
tags = ["productivity", "projects"]
draft = false
math = true
+++

This quarter, I want to do several "one-off" projects; this is the first of them.

This project is primarily inspired by ["100 Blocks a Day"](https://waitbutwhy.com/2016/10/100-blocks-day.html) idea put forth by Tim Urban. The basic idea is that we are awake for roughly 1,000 minutes a day; thus, you can frame one day as consisting of 100 10-minute blocks. Then, off of that you can think about all sorts of things like...how many blocks do I have left? How many more blocks will I spend with family, will I spend scrolling social media, will I spend in deep conversations with friends, will I spend deeply immersed in stuff that I care about?

So, essentially I seek to build upon this idea by tracking what I'm doing each 10-minute "block" of the day. After building up some blocks, (say, at the end of the day) during my daily review, I can review my 100 blocks for the day, and see where my time really went. Same thing could be done for months, years, and lifetimes. I think that this provides a few benefits:

1.  **Objective view of myself.** It's hard to really know how I'm doing in life, and reflect and improve on that, without having hard data on _what_ I'm doing.
2.  **Increased mindfulness**. The 10-minute reminders not only passively track me, but also double as a mindfulness reminder. If I've been mindlessly scrolling on Reddit or perseverating over some insignificant worry, the reminder serves as a nudge to say, "Hey, a block of time just went by," and gently redirect my attention back to the task at hand.


## Part 1: Polling myself every 10 minutes {#part-1-polling-myself-every-10-minutes}

The first component is a mechanism to poll myself every 10 minutes. But first, I had to ask myself a couple of questions to figure out how exactly I want to make this polling mechanism:


### How much granularity to have in distinguishing activities? {#how-much-granularity-to-have-in-distinguishing-activities}

Well, to answer that, I have to ask "What do I want to get out of this?" Well, one major thing is I want to see how much (contiguous) time I spend in [Deep Work](https://blog.doist.com/deep-work/). Also, I would like to see how much time I'm spending with my family, my girlfriend, my exercise routine, etc. So really, it doesn't need to be a detailed log -- can just be broad categories, such as Projects, Writing (such as I'm doing right now,) Social Time, Eating, Exercise, and Sleeping (for my own sanity, I will not be waking up every 10 minutes to track my sleep...I let [Sleep Cycle](https://www.sleepcycle.com/) take care of that for me.)


### What platform to make the polling mechanism on? {#what-platform-to-make-the-polling-mechanism-on}

One option might be an iOS app -- this feels like a bit overkill, though, because I really don't need a custom user interface -- I just need text inputs from myself at regular intervals. I want this polling to be as unintrusive as possible -- a very simple click/tap or two every ten minutes, and I'm on my way. I've experimented with simply using the Reminders app and Google Sheets -- however, this procedure is rather cumbersome, because the reminder notification itself doesn't lead directly to the spreadsheet, and scrolling down and entering values in a spreadsheet is annoying. This procedure would take me around 40 seconds -- which is essentially taking away nearly 10% of my time.

Given these constraints, Messenger bots seem to be a good platform -- the front-end UI is already done for me, and I can simply program a bot to ask me "What are you doing?" every 10 minutes. I can even include nice features like Quick Replies, so I can supply input with one tap.


### Building a Messenger bot {#building-a-messenger-bot}

First, I followed the instructions in <https://developers.facebook.com/docs/messenger-platform/getting-started> to set up the Facebook Page and starter code for the Messenger bot's _webhook_, to which all messages from the user will be routed to and from which all bot replies are sent. I chose to deploy my webhook to Heroku.

I took the code from [this sample Messenger app](https://github.com/fbsamples/messenger-platform-samples/tree/master/quick-start) to start out, which defines the necessary routes for the webhook, providing some useful helpers to respond to user messages, use the [Send API](https://developers.facebook.com/docs/messenger-platform/reference/send-api), etc.. . I modified the `handleMessage` function so that the initial message from the user starts a chain of repeating messages, 10 minutes apart, utilizing the `setInterval` JS function to schedule quick-replyable messages to the user. This makes following flow possible:

![Demonstrating the "quick reply" workflow in Messenger.](/ox-hugo/quick_reply_workflow.gif)

After receiving the activity input from the user, I add a record to DynamoDB:

```js
function addActivityToDatabase(activity) {
  const timestamp = new Date().toISOString();
  var params = {
    TableName : DYNAMODB_TABLE_NAME,
    Item: { activity, timestamp }
  };
  ddb.put(params, function(err, data) {
    if (err) console.log(err);
    else {
      console.log(`Successfully put data ${JSON.stringify(params.Item)} in DynamoDB!`)
      console.log(data);
    }
  });
}
```

So now that we have the tracking flow working and storing data, that brings us to the next section...


## Part 2: Visualizing my time in daily review {#part-2-visualizing-my-time-in-daily-review}
