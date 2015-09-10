---
layout: post
title: "Another jQuery cautionary tale"
description: ""
category:
tags: [javascript]
---
{% include JB/setup %}

One second, [I need to compose myself.](http://espn.go.com/blog/new-york/mets/post/_/id/110476/mets-sweep-nationals-in-catbird-seat-in-nl-east-race)

There are plenty of warnings out there that reliance on undocumented behavior is dangerous and could potentially break your app. Of course, there are many more developers out there that will still manage to sneak in a hack every now and then that will take advantage of some quirk which may or may not come back to haunt them later. The latest one of these I found had to do with jQuery's [.text()](http://api.jquery.com/text/) function and its behavior across versions.

<!--more-->

While refactoring a portion of our site we upgraded from jQuery 1.6.4 to 1.11.0. During QA it came to my attention that there was some random spacing appearing between the some validation error messages that did not happen prior to the upgrade.

<div class="row">
<div class="col s12 m4 offset-m2">
<div class="card" style="max-width:200px;margin: 5px auto;">
    <div class="card-image">
        <img src="/assets/images/2015-09-10-error-messages.png">
    </div>
    <div class="card-content">
        <p>Normal error messages</p>
    </div>
</div>
</div>

<div class="col s12 m4">
<div class="card" style="max-width:200px;margin: 5px auto;">
    <div class="card-image">
        <img src="/assets/images/2015-09-10-error-messages-spaced.png">
    </div>
    <div class="card-content">
        <p>Slight vertical spacing between DOB and Email errors</p>
    </div>
</div>
</div>
</div>

*Great, one of these.* I opened up my dev tools to see what was going on.

<img style="max-width:250px;" src="/assets/images/2015-09-10-blank-error-div.png">

Without changing any script related to error message construction, a blank error div appeared out of the nether that wasn't there before. I dug deeper into the third-party script we fetched server-side that dealt with these messages and found the relevant code:

<pre class="prettyprint lang-js">
for (var field in cfg.errorFields) {
  if ((field === "month" || field === "day" || field === "year") && !dobAdded) {
    innerShell.append($("<div class=\"error\">").text(self._blankMessages["dateOfBirth"]));
    dobAdded = true;
  } else {
    innerShell.append($("<div class=\"error\">").text(self._blankMessages[field]));
  }
}
</pre>

So if any one of the "month", "day", or "year" fields were blank, the code would show the message for DOB. On a subsequent iteration, if it caught another one of these 3 fields which was also blank, it defaulted to appending a blank error div and assigning a default message for that particular field. Except "month", "day", and "year" were not indices of self._blankMessages, so the argument defaulted to `undefined`. On the old page, the result was no div being constructed. Why did it start appearing after the upgrade?

My suspicions were confirmed after running the same code on a test page with two different jQ libraries:

<pre class="prettyprint lang-js">
> jQuery.fn.jquery
"1.6.4"
> $('&lt;div class="error">').text(undefined)
""
</pre>

<pre class="prettyprint lang-js">
> jQuery.fn.jquery
"1.11.0"
> $('&lt;div class="error">').text(undefined)
[ &lt;div class="error"></div> ]
</pre>

Yep, there it is. Let's take a closer look at what's going on in jQuery:

<pre class="prettyprint lang-js">
// jQuery 1.6.4
jQuery.fn.extend({
  text: function( text ) {
    if ( jQuery.isFunction(text) ) {
      return this.each(function(i) {
        var self = jQuery( this );

        self.text( text.call(this, i, self.text()) );
      });
    }

    if ( typeof text !== "object" && text !== undefined ) {
      return this.empty().append( (this[0] && this[0].ownerDocument || document).createTextNode( text ) );
    }

    return jQuery.text( this ); // In our case "text" is undefined, so getter function
                                // is triggered which will return a blank string
  }

  //...
});
</pre>

<pre class="prettyprint lang-js">
// jQuery 1.11.0
jQuery.fn.extend({
  text: function( value ) {
    return access( this, function( value ) {
      return value === undefined ?
        jQuery.text( this ) :
        this.empty().append( ( this[0] && this[0].ownerDocument || document ).createTextNode( value ) );
    }, null, value, arguments.length );
  }

  //...
});
</pre>

Both versions check if the value is defined in order to determine if it should invoke getter or setter behavior. However 1.11.0 makes use of a separate function called `access` to do this. This function appeared sometime between 1.6.4 and 1.7.2.

<pre class="prettyprint lang-js">
var access = jQuery.access = function( elems, fn, key, value, chainable, emptyGet, raw ) {
  var i = 0,
    length = elems.length,
    bulk = key == null;

  // Sets many values
  if ( jQuery.type( key ) === "object" ) {
    chainable = true;
    for ( i in key ) {
      jQuery.access( elems, fn, i, key[i], true, emptyGet, raw );
    }

  // Sets one value
  } else if ( value !== undefined ) {
    chainable = true;

    if ( !jQuery.isFunction( value ) ) {
      raw = true;
    }

    if ( bulk ) {
      // Bulk operations run against the entire set
      if ( raw ) {
        fn.call( elems, value );
        fn = null;

      // ...except when executing function values
      } else {
        bulk = fn;
        fn = function( elem, key, value ) {
          return bulk.call( jQuery( elem ), value );
        };
      }
    }

    if ( fn ) {
      for ( ; i < length; i++ ) {
        fn( elems[i], key, raw ? value : value.call( elems[i], i, fn( elems[i], key ) ) );
      }
    }
  }

  return chainable ?
    elems :

    // Gets
    bulk ?
      fn.call( elems ) :
      length ? fn( elems[0], key ) : emptyGet;
};
</pre>

So what's changed? In 1.6.4 an `undefined` argument triggered the getter functionality of .text(), which was actually an [issue](https://github.com/jquery/jquery/pull/392) of some debate years ago. With the addition of the access function however, a new condition `chainable` was introduced. In a typical getter invocation, .text() will call `access` as follows:

<pre class="prettyprint lang-js">
return access( this, function( value ) {
  return value === undefined ?
    jQuery.text( this ) :
    this.empty().append( ( this[0] && this[0].ownerDocument || document ).createTextNode( value ) );
}, null, value, arguments.length ); <span class="bold">// value = undefined, arguments.length = 0</span>
</pre>

Since `arguments.length` is 0, `chainable` would evaluate to false and the callback provided from `text`, which switches on `return value === undefined` and runs `jQuery.text( this )`, would be invoked via `fn.call( elems )`.

According to the issue above, passing .text(undefined) should invoke the getter as well -- except we never get there. Since passing actual `undefined` results in `arguments.length` evaluating to 1, even though `value` is the same in both scenarios the callback function is never invoked! Instead, `elems` is returned rather than the blank string from the result of `jQuery.text`, so that was why the blank error div was being generated in the new site.

As of now I'm not sure if this modified behavior was an intended side effect of jQuery's access function or not, but there is a more important lesson here. The code should have been more explicit in handling this kind of scenario, especially since the fix is very simple:

<pre class="prettyprint lang-js">
for (var field in cfg.errorFields) {
  if (field === "month" || field === "day" || field === "year") {
    if (!dobAdded) {
        innerShell.append($("<div class=\"error\">").text(self._blankMessages["dateOfBirth"]));
        dobAdded = true;
    }
  } else {
    innerShell.append($("<div class=\"error\">").text(self._blankMessages[field]));
  }
}
</pre>

Problem solved. Instead of defaulting to the else case when another DOB field is encountered, simply skip adding a DOB error div if we already did so. Relying on a side-effect of passing `undefined` is never a good idea, and it will eventually create a headache for your coworkers.

I understand this kind of stuff happens -- after all, deadlines exist -- but I hope this will remind you (it certainly did for me) to stop and think for a second next time you're in a situation like this and evaluate whether an odd behavior may be subject to change in the future.