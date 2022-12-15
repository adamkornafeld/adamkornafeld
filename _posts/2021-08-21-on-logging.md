---
layout: article
title: The one that suppresses log messages
tags: java log suppress user-interface
mode: immersive
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: '#203028'
  background_image:
    src: https://wallpaperaccess.com/full/521273.jpg
---

In the [previous post](https://kornafeld.com/2021/04/10/on-naming.html) we have implemented a `Clock` that is useful for deterministic testing of logic that depends on the passing of time. In this post, we will take a look at logic that is driven by time.


✨ Connecting the dots
---------

In his 2005 Stanford Commencement Address, Steve Jobs famously told the story of [connecting the dots](https://youtu.be/UF8uR6Z6KLc?t=55). I feel lucky to have been able to connect some dots myself. When my focus shifted from mobile app development to backend coding, the lack of user interfaces were somewhat frustrating in the beginning. But soon, it dawned on me that there is indeed a user interface that most backend engineers will be staring at once their code is deployed in production. Whether this user interface is helpful and a joy to use, or a useless nightmare depends on us, engineers. This user interface is the log stream. Besides the API that your backend code connects to the outside world with, the stream of log messages will be your only window into the inner workings of your code once it has been deployed. And oh, you will be staring at it wondering what on earth is your code doing on a late-late Sunday night. Enough intro, let's do some coding, shall we?


🍃 The fallacy of log messages
---------

Take a look at this piece of code:

    if (customer.exists(customerId)) {
        customerRepository.delete(customerId);
        log.info("{} deleted", customerId);
    }

Perfect logic, right? If the customer exists, we delete it. What's wrong here, you might ask? Let's say that the type of `customerId` is number and its value for the sake of this example is 5. The log message will rightfully read: '5 deleted'. Is that useful? Well, it might be if customer is the only type of entity in your system that can be deleted. If, however, there is also a product entity or god forbid a few hundred other entities in your system this message will not prove too useful. You might call me out here saying, but look at the code, it's super obvious that we are deleting a customer. And you are right as long as you have the source code in front of you. And that is a fallacy I see quite a few backend engineers fall into. At the time of writing the business logic, everything is super logical in their head, so the amount of information they will put into the log message will be minimal. Detrimental even, if you have to figure out a bug on a Sunday night and all you have is this log message. But no source code.

You might ask, how can we _fix_ the code above? Well, for starters we can include the type of entity in the log message:


    if (customer.exists(customerId)) {
        customerRepository.delete(customerId);
        log.info("Customer({}) deleted", customerId);
    }

This would yield a message like 'Customer(5) deleted'. Much better, but thinking about this with a user experience perspective we can take this one step further:

    if (customer.exists(customerId)) {
        customerRepository.delete(customerId);
        log.info("Customer(customerId={}) deleted", customerId);
    }

Your 'sleepy, on-call self' will thank me at 3 am on a Sunday-night after reading the log message `Customer(customerId=5) deleted` and knowing exactly what's going on here. Also, please invest in writing self-documenting code. One way of doing that is strategically placing debug messages, like so:

    if (customer.exists(customerId)) {
        customerRepository.delete(customerId);
        log.info("Customer(customerId={}) deleted", customerId);
    } else {
        log.debug("Customer(customerId={}) does not exist, nothing to delete", customerId);
    }


💦 The waterfall of log messages
---------

Another issue with log messages that I ran into before is that an ill-placed log message in a loop or a logic that gets called often floods the log stream. This renders it nigh impossible to extract anything useful from it. You suddenly feel like having to find the needle in the haystack. The simple solution to this is of course _'let's just not put log inside loops'_. However, that's not always possible. Often times, at development time it might not even be obvious that the log message you implement will end up being executed frequently. So what can we do, if we find ourselves in a situation that we would like to see the log message, but also not flood the log stream with it? 


🪵 Log suppressor
---------

You might be lucky, that the log library you use provides support for that. I wasn't, so let's implement a log suppressor. A small utility class that you can wrap your log messages with and have it keep track of time to determine if it should log the message or swallow it. Naming our class is fairly straightforward this time. Call it `LogSuppressor` and feel lucky that we found a suitable name for our class on first try.

    class LogSuppressor {
       
        private final Duration interval;

        public LogSuppressor(Duration interval) {
            this.interval = interval;
        }
    }

By passing an `interval` at construction time, our class will emit only one log message per interval. The interval is fully configurable, from minutes, to days, even weeks. You name it. We also need to keep track of how much time has passed since a log message was last emitted. For that we will maintain an internal instance of `Instant`. We can initialize it with `Instant.MIN` to signal that our log suppressor has not suppressed anything yet.

    class LogSuppressor {
       
        private final Duration interval;
        private Instant suppressedAt;

        public LogSuppressor(Duration interval) {
            this.interval = interval;
            this.suppressedAt = Instant.MIN;
        }
    }

With those two properties in place we can implement the main function of our utility class: `LogSuppressor::suppress`

    public boolean suppress(Runnable function) {
        Instant now = Instant.now();
        if (now.isAfter(suppressedAt)) {
            function.run();
            suppressedAt = now;
            return false;
        }
        return true;
    }

That should be straightforward, we look at the current _instant_ and see if it is after the instant when our log suppressor last suppressed a message. On first run, the condition will be true as the current instant of our clock will be after `Instant.MIN` and we run the function that we received as an input. This function wraps a log message. We also update the instant value of `suppressedAt` and return false signaling that this time the suppressor did not suppress. In all other cases, we return true. Let's look at a usage example:

    LogSuppressor dailySuppressor = new LogSuppressor(Duration.ofDays(1));

    dailySuppressor.suppress(() -> log.info("This message will be logged once a day"));

Two nuances to recognize. One, restarting the app during the day will cause the log message to be emitted again, so it is possible for our log suppressor to log more than once a day. If need be, some persistence can be added to deal with that. Second, the input argument of the `suppress` function is a generic function. It is not limited to implement only log messages. You can technically use `LogSuppressor` to suppress any kind of business logic. My gut feeling, though, is that there might be side effects there, so I opt not to generify `LogSuppressor` as something like `FunctionSuppressor`.


🪵 Testing log suppressor ⏱
---------

To frame this post, we can make one quick improvement by using the `ToyClock` from the [previous post](https://kornafeld.com/2021/04/10/on-naming.html). See, the passing of time is at the heart of our `LogSuppressor` and writing a deterministic unit test for it could be a challenge. Unless of course we have `ToyClock` with which we can tightly control the ticking of time. So let's inject a clock into `LogSuppressor`


    class LogSuppressor {
       
        private final Duration interval;
        private Instant suppressedAt;
        private final Clock clock;

        public LogSuppressor(Duration interval, Clock clock) {
            this.interval = interval;
            this.suppressedAt = Instant.MIN;
            this.clock = clock != null ? clock : Clock.systemUTC();
        }

        public LogSuppressor(Duration interval) {
            this(interval, Clock.systemUTC());
        }

    }

There was a _buy one get one free_ deal, so we threw in a convenience constructor as usually we will want to rely on the system clock. There is only one change we need to make to the implementation of the `suppress` function. That is inject the clock into `Instant::now`. 

    Instant now = Instant.now(clock);

With the clock in place, testing the logic becomes a breeze:

    @Test
    void suppress() {
        Clock twentyOneSecondTickingClock = new ToyClock(Duration.ofMinute(21));
        LogSuppressor minuteSuppressor = new LogSuppressor(Duration.ofMinute(1), twentyOneSecondTickingClock);
        // every call to suppress ticks the toy clock injected into LogSuppressor once
        
        minuteSuppressor.suppress(() -> log.info("This message will be logged")); // clock advances 21 seconds
        minuteSuppressor.suppress(() -> log.info("This message will be suppressed")); // clock advances to 42 seconds, one minute has not elapsed yet, log is suppressed
        minuteSuppressor.suppress(() -> log.info("This message will be logged")); // clock advances 63 seconds
    }

And that wraps our coding session for today. Happy coding!
