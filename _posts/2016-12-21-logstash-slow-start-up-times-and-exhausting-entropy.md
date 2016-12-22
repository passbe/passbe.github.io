---
title: Logstash slow start-up times and exhausting entropy
layout: post
excerpt: "I spent the better part of two hours debugging a non-responsive Logstash instance. Turns out it was just slow and /dev/random was the culprit."
---

Recently I have been working with the [ELK](https://www.elastic.co/webinars/introduction-elk-stack) stack to develop some training. I had provisioned a small VM to run all the software required and Logstash was happily ingesting a small amount of logs.

I started to fiddle with the [Logstash aggregate filter](https://www.elastic.co/guide/en/logstash/current/plugins-filters-aggregate.html) when all of a sudden Logstash became completely unresponsive. No logs, no warnings or errors, yet systemd told me it was happily running.

Two hours later I had completely reinstalled Logstash only for the problem to persist. In despair I sat back for a few minutes and suddenly logs started flowing.

![Logstash - Y U NO WORK?](/assets/2016/logstash-meme.png "Logstash - Y U NO WORK?")

Turns out Logstash was taking ~5 minutes to start. After a bit of research I found [this Github issue](https://github.com/elastic/logstash/issues/5507) which directed me to the [JRuby wiki](https://github.com/jruby/jruby/wiki/Improving-startup-time#ensure-your-system-has-adequate-entropy) detailing the issue:

> When JRuby boots up, the JDK libraries responsible for random number generation go to /dev/random for (at least) initial entropy. After this point, more recent versions of JRuby will use a PRNG for subsequent random numbers, but older versions will continue to return to /dev/random. Unfortunately /dev/random can "run out" of "good" random numbers, providing a guarantee that reads from it will not return until the entropy pool is restored. On some systems -- especially virtualized -- the entropy pool can be small enough that this slows down JRuby's startup time or execution time significantly.

Note the words "*especially virtualized*". To see your available entropy, run:

```
cat /proc/sys/kernel/random/entropy_avail
```

Anything less than 1000 is sub-optimal. In my case it came back with 54!

One solution is to install the **haveged** package. Once installed Logstash should start in a reasonable time. Two hours wasted, hopefully this post will help a few people.

##### References

* [Github Issue - Why is Logstash so slow to start?](https://github.com/elastic/logstash/issues/5507)
* [JRuby Wiki - Improving startup time](https://github.com/jruby/jruby/wiki/Improving-startup-time#ensure-your-system-has-adequate-entropy)
* [ArchLinux Wiki - Haveged](https://wiki.archlinux.org/index.php/Haveged)
* [Digital Ocean - How to Setup Additional Entropy for Cloud Servers Using Haveged](https://www.digitalocean.com/community/tutorials/how-to-setup-additional-entropy-for-cloud-servers-using-haveged)
