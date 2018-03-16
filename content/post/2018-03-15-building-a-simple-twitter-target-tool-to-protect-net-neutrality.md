---
title: Building a simple Twitter target tool to protect Net Neutrality in Canada
author: Vojtech Sedlak
date: '2018-03-15'
slug: building-a-simple-twitter-target-tool-to-protect-net-neutrality in Canada
categories:
  - Hacks
tags:
  - JavaScript
  - jQuery
  - Google Spreadsheets
  - Campaigns
---

![Unfairplay.ca](/img/post-unfairplay.png)

A couple of weeks ago a friend and an accomplished campaigner, Josh Tabish ([@jdtabish](https://twitter.com/jdtabish)), came up with the idea of building a simple page that would allow people to Tweet at companies in the FairPlay coalition, which proposed to censor Internet in Canada. It turned out to be a fun little project, resulting in [https://unfairplay.ca](https://unfairplay.ca). 

The site got a lot of attention, gaining a top spot on /r/technology, getting over 60,000 visits, generating 17,000 tweets and some 5,000 comments to the CRTC. Since building such a tool could be a helpful tactic for other groups I wanted to share the code, so you can leverage the power of Twitter yourself. 

In case you haven't already heard about the issue, a group of Canadian companies, spearheaded by none other than Bell, recently put forward a proposal to introduce Internet censorship as a solution to piracy. Not only does it violate the basic principles of open Internet, it is a potentially dangerous proposal that lacks proper judicial oversight and could seriously affect the future of Internet in Canada. You can read all about it at Michael Geist's [blog](http://www.michaelgeist.ca/2018/02/canadas-sopa-moment-crtc-reject-bell-coalitions-dangerous-internet-blocking-plan/) or [here](https://www.theglobeandmail.com/report-on-business/anti-piracy-group-under-fire-for-website-blocking-proposal/article37856303/).

## Why use a spreadsheet

The original inspiration for this tool came from FightForTheFuture, who used something similar in the [Battle For The Net](https://battleforthe.net). In their case they were targeting US policymakers, but you can go after anyone, who has a Twitter account. Yes, you could just hardcode the Tweets, but if you have more than 10 targets, or want to customize the Tweets, that may get a bit unwieldly. That's where Google Spreadsheets come in. Instead of having to spin up a database, Google Spreadsheets are a simple of way of storing and accessing columnar data.

Start by assembling a list of your targets, their Twitter handles and the tweet copy. You can also add a name of an associated image file, if you want to include their portrait/logo. You can see an example of the spreadsheet we used below.

![spreadsheet](/img/post-spreadsheet.png)

In order to be able to access the spreadsheet data programatically, you will need to make the spreadsheet public using the _Publish to Web_ feature. That will make the data accessible to anyone, so it's not a good idea to include any sensitive data in the spreadsheet.

![publish](/img/post-publish.png)

## Accessing the data with jQuery

Once your spreadsheet is populated with data and is publish to the web, you can access it programatically. For Unfairplay.ca we built a simple static site that used the ```$.ajax()``` jQuery function to dynamically add Tweet buttons for all targets. You can see the full code below:

```
$(document).ready(function() {
    $.ajax({url: "https://spreadsheets.google.com/feeds/list/1J9RVuxRU_5MDO1K0FAucTNN0tvf5N7XVQuDH7TR9Bgg/default/public/values?alt=json", success: function(result){
        var data = result.feed.entry;
        for (var i=0;i<data.length;i++) {
        	var name = data[i].gsx$organization.$t;
        	var logo = data[i].gsx$imagepleasedontedit.$t;
        	var handle = encodeURI(data[i].gsx$twitter.$t);
        	var tweet = encodeURI('.'+handle+': Withdraw your support for @FairPlayCanada. This ineffective Internet censorship proposal will harm consumers, innovation and free expression online (cc: @CRTCeng @NavdeepSBains) https://unfairplay.ca')
        	$('#board').append('<div class="entity col-3"><div class="entity-image"><img src="assets/images/'+logo+'"></div><a target="_new" onClick="gtag(\'event\', \'Tweet Click\')" class="btn" href="https://twitter.com/intent/tweet?text='+tweet+'&hashtags=DontCensor"><img src="assets/images/twitter_white.svg" width:20px;"> Tweet</a></div>');
        }
    }});
})
```

The code simply accesses the spreadsheet feed in json (don't forget to add ?alt=json at the end of the URL) and for each row parses the different values (name, logo and handle) and then appends a ```div``` with a link that includes the dynamically generated elements. For Unfairplay.ca we ended up using a stock Twitter copy, but you could easily have a column for specific Tweets in the spreadsheet.

Things to keep in mind if you are adapting the script:

- The basic structure of the URL is ```https://spreadsheets.google.com/feeds/list/[[ID OF THE SPREADSHEET]]/default/public/values?alt=json```. All you need to do is add the ID, which you find in the address bar of the spreadsheet URL.
- A column name is used to access the given values (```data[i].gsx$organization.$t```). Use console.log to double check you are accessing the right values.
- Since you are building a URL with text strings from the spreadsheet, use ```encodeURI()``` to encrypt all non-standard characters. Otherwise your Tweets may be mangled.
- If you are using images, you will need to upload them separately to a given folder and ensure they all have similar dimensions. It may take some CSS hacks to make things look alright.

## Striking a chord

The [Unfairplay.ca](https://unfairplay.ca) site got some attention on Canadian subreddits, but it wasn't until someone posted in on /r/technology that things really accelerated. In one day over 50k people visited the site, sending 16k tweets at the target companies. At one point the site even held the top post on the /r/technology subreddit, with over 39,000 upvotes! Not bad for a couple hours of work.

![reddit](/img/post-reddit.png)

You can find the repository for the Unfairplay.ca site [here](https://github.com/vojtechsedlak/unfairplay). **Kudos to Josh for the great idea and to OpenMedia for hosting the page, so that it didn't crumble when it hit reddit.** If you have any ideas for how to improve this, feel free to add them below.