If you're like me, you heard the term "dependency injection" (probably in the Java world) and immediately rushed over to StackOverflow fearing you would immediately lose your first full-time job for not having ever heard the term prior to graduation. It certainly *sounds* complex, and if you had the pleasure of working with a framework like [Spring](http://projects.spring.io/spring-framework/) you may have been confused even further with its annotation-based way of doing things.

Relax. At its core it's a pretty simple concept &mdash; just ask [James Shore](http://www.jamesshore.com/Blog/Dependency-Injection-Demystified.html):

> *"Dependency Injection" is a 25-dollar term for a 5-cent concept.*

If you've searched around SO or other resources (or read the one I just linked) and still haven't gotten a handle on the concept quite yet (or have but are not sure why it is useful), hopefully I can help. In fact, I'll explain it using a real-life issue that came up at work recently rather than try to utilize another generic [animal/person/car example](https://twitter.com/iamdevloper/status/531570316037087232).

When a user visits our website, we always want to know what kind of device they are using. There are various reasons for this: determining the optimal site layout for the user's screen, recording analytics, things like that. To determine this, our server makes a call to a DeviceAnalyzer service, which takes information from the request and returns what eventually ends up being a string containing either "DESKTOP", "TABLET", or "MOBILE". This string will help to instruct the server what kind of decisions to make regarding the above functionality. 

One important thing to note is that *if our server already knows what device we are dealing with prior to calling DeviceAnalyzer, **it skips the call altogether***.

So far so good. Except one day, the team received an email complaining about unnecessary requests being made to the DeviceAnalyzer service when it wasn't needed. We found that this occured when we ran cron jobs to do various tasks, ranging from building cache files to processing incomplete tasks that were stored in the database to be handled later. If you're not familiar with how a cron job works then simply picture the issue this way: we were trying to determine the device for a request that did not have any device associated with it, i.e. a request from a command line.

So I started to dig through the code. I knew in all likelihood the cron job was using a library that was also used in servicing requests made by normal site users, so something inside that library was trying to check which device the request came from. Eventually I found myself looking at this snippet (some variable names changed, but the idea is the same):

``` php
public static function getObjectToProcessTask ($user) {
    if (is_null(self::$objectToProcessTask)) {
        //...
        $objectToProcessTask = new ObjectToProcessTask();
        $objectToProcessTask->setSourceDevice();
        self::$objectToProcessTask = $objectToProcessTask;
        //...
    }
    return self::$objectToProcessTask;
}
```

*Okay,* I thought. *So we're getting an instance of an object that will clearly do something later that will require calling DeviceAnalyzer. Why don't we just give it some dummy device from the cron job, so it knows what the device is and skips the call to DA?*

Well sure enough, that's what the setter **`setSourceDevice`** method of the `ObjectToProcessTask` class was supposed to do:

``` php
public function setSourceDevice ($device=null) {
    if ($device != null) {    
        $this->sourceDevice = $device;    
    }
    else {
        $deviceType = strtoupper(DeviceAnalyzer::getType()); // if $device is not supplied, call DeviceAnalyzer
        $this->sourceDevice = $deviceType;        
    }
}
```

In order for the class methods to do what they needed to do, they depended on the **`$sourceDevice`** property having some sort of value. However, instead of the class always automatically assuming we didn't have this information and making a call to DA, this setter gives us the opportunity to pass in the device beforehand (for example, some dummy value like 'cron'), so when another method is invoked the class will already have the device at its disposal and skip the DA call.

Why then did the first snippet have this line?

``` php
$objectToProcessTask->setSourceDevice();
```

Our only way of getting this object is using the `getObjectToProcessTask method`, and it's just assuming we don't have a device to give it! Instead, the function should be written like this:

``` php
public static function getObjectToProcessTask ($user, <span class="highlight-code">$device=null</span>) {
    if (is_null(self::$objectToProcessTask)) {
        //...
        $objectToProcessTask = new ObjectToProcessTask();
        $objectToProcessTask->setSourceDevice($device);
        self::$objectToProcessTask = $objectToProcessTask;
        //...
    }
    return self::$objectToProcessTask;
}
```

Perfect! Now we can actually use that setter in the way it was intended &mdash; to inject the device if we so choose instead of making its own decision and wasting a call to DeviceAnalyzer. In the cron script we can now do the following:

``` php
ObjectToProcessTaskHelper::getObjectToProcessTask($user, 'cron');
```

Sure enough, the extra requests to DeviceAnalyzer went away after that. We could even take it a step further (and in the Java world this would almost certainly be the case) and use an interface for the device:

``` php
public static function getObjectToProcessTask ($user, IDevice $device=null) {
    //...
}
//...
class DesktopDevice implements IDevice {...}
class CronDevice implements IDevice {...}
```

Since the argument is an interface, any mock device object can be used.

So what have we learned from this?

* Dependency injection is just passing in values a class depends on rather than it assuming some default value or behavior. We are *injecting* whatever properties the class requires for use in later functionality. We can achieve this a couple ways:

  * A mutator (setter) method, like we saw with setSourceDevice()
  * Arguments to a constructor (also kind of what we saw above; we ended up passing a value to a method which constructed the object if it did not exist, which in turn used it in a setter).

* DI gives us the flexibility to swap out values without having to rewrite code. If we wanted to run a test suite against this module, we could just as easily pass in 'test' as the `$device` argument instead of 'cron' and achieve the same end result.

That's really all there is to it. There's no extra magic &mdash; all those complex frameworks employ various methods that eventually boil down to this simple concept.