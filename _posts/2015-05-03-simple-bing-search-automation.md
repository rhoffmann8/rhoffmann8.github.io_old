---
layout: post
title: "Simple Bing search automation"
description: ""
category: 
tags: [javascript]
---
{% include JB/setup %}

After having a bit of trouble figuring out how I would start off the first post, I decided to start with something very simple, yet useful to me. 

Not too long ago it occurred to me I could be getting free Amazon cards from [Bing Rewards](https://www.bing.com/rewards) for just running the same searches every day. After kicking myself for not realizing this sooner, I went to work looking for automation solutions.
<!--more-->

Of course various automation sites and extensions exist for achieving just this, but I rolled my own solution because:

* I like knowing exactly what the tool is doing
* There likely is educational value, however small, in coding your own solution

The result was [this small piece of JS](https://github.com/rhoffmann8/autobing) which I run in the browser console every day:

<pre class="prettyprint lang-js">
var xhr = new XMLHttpRequest(),
    multiplier = 1000, // 1 second
    topics = [ // some common topics
        'weather', 'news', 'sports', 'actors', 'television', 'movies', 'economy', 'business', 'politics', 'music', 
        'games', 'food', 'cars', 'fashion', 'clothing', 'trending', 'microsoft', 'google', 'facebook', 'twitter', 
        'wikipedia', 'reddit', 'web', 'java', 'javascript', 'ecmascript', 'nodejs', 'jquery', 'angularjs', 'coffeescript', 
        'npm', 'karma', 'grunt', 'gulpjs', 'bower', 'php', 'python', 'ruby', 'c++', 'github'
    ];

for (var i = 0; i < topics.length; i++) {
    (function(time) {
        setTimeout(function() {
            xhr.open('GET', window.location.protocol + '//www.bing.com/search?q='+topics[time]+'&go=Submit');
            xhr.send();
        },time*multiplier);
    })(i);
}
</pre>

Mobile searches can be done by simply toggling device emulation in Developer Tools.

To make things even easier I found a [Chrome extension](https://chrome.google.com/webstore/detail/shortcut-manager/mgjjeipcdnnjhgodgjpfkffcejoljijf) which allows for mapping keyboard shortcuts to JS code. I simply added Bing to the shortcut's matching URL patterns and mapped a key combination to execute the JS code located on my Github repo.

And that's all there is to it! If you're a bit perplexed by the function invocation in that loop up there, take a look at the [loop-closure problem](https://www.google.com/?gws_rd=ssl#q=javascript+loop+closure+problem) in JavaScript.