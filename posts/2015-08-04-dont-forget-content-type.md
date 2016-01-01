After two months of [disappointment](http://www.nydailynews.com/sports/baseball/mets/mets-anemic-offense-degrom-stop-cubs-6-1-loss-article-1.2279653) [and](http://espn.go.com/blog/new-york/mets/post/_/id/105410/terry-collins-no-batting-help-left-in-minors-for-anemic-mets) [frustration](http://nypost.com/2015/06/07/mets-anemic-offense-wastes-colons-gem-in-loss-to-dbacks/), things are finally [looking](http://www.newsday.com/sports/baseball/mets/mets-hit-three-home-runs-in-span-of-five-pitches-in-5-2-win-over-nationals-1.10701085) [up](http://sports.yahoo.com/blogs/mlb-big-league-stew/yoenis-cespedes-powers-red-hot-mets-into-sole-possession-of-first-place-063140941.html) for us Mets fans, aren't they? [Bryce might disagree though.](http://espn.go.com/blog/new-york/mets/post/_/id/108124/bryce-harper-gets-snippy-after-mets-take-sole-possession-of-first)

This isn't so much a code tip as it is a friendly reminder to remember to specify your `Content-Type` header when making a HTTP request. Due to this, I recently ran into a interesting little issue at work.

We have a small web-based tool that allows developers to upload HTML5 games and test them against a dummy API. Over the past few days I was tasked with recreating the tool in Node.js (it's an old PHP-based page that needed some fixing up -- I reached the conclusion it would be much simpler to redo it). At certain points during play, the game would send a JSON request with various details: event type, token, game data, etc. To receive and process the request, I had a pretty standard-looking [express](http://expressjs.com/) route using a JSON parser:

``` js
var express = require('express');
var bodyParser = require('body-parser');

var app = express();
app.use(bodyParser.json());

//...

app.post('/events', function(req, res) {
  log(req.body);
  processEvent(req.body);

  res.end();
});
```

However, every time I triggered the appropriate action in the provided HTML5 game to send the request, `req.body` was empty! As an intermediate Node.js programmer I thought I had missed something simple or perhaps was using some deprecated parsing functionality by mistake. I checked the request payload to see if it was syntactically incorrect, but to no avail. After about a half hour of futility it occured to me to double check the request headers:

<div class="row center">
  <div class="img-holder">
    <div class="img-holder-inner">
      <img src="/images/2015-08-04-content-type.png" style="width:280px;">
    </div>
    <p class="caption">
      Content-Type here is "text/plain" instead of "application/json"
    </p>
  </div>
</div>

The game was not setting the `Content-Type` header to **application/json** -- instead it was **text/plain**. This made the bodyParser unable to parse it correctly.

In a perfect world this would be corrected in the game itself by setting the header correctly, but this being reality I knew I would not be able to fix the other hundreds of games that had the same issue. So, I had to settle for parsing the request in its raw form:

``` js
function getRequestPayload(req) {
  return new Promise(function(resolve, reject) {
    var str = '';
    req.on('data', function(buf) {
      str += buf;
    });
    req.on('end', function() {
      resolve(str);
    });
  });
}

app.post('/events', function(req, res) {
  getRequestPayload(req)
    .then(function(json) {
      var json = JSON.parse(json);
    
      log(json);
      processEvent(json);

      res.end();
    })
    .catch(function(err) {
      log(err);
      res.end();
    });
});
```

If you're not familiar with [promises](https://promisesaplus.com/) you should definitely check them out. In this particular instance I'm using the [Bluebird](https://github.com/petkaantonov/bluebird) promise library.

Hopefully this reminder will spare another developer the expense of having to work around this.