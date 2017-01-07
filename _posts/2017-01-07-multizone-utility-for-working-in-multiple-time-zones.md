---
title: Multizone - a utility for working in multiple time zones
layout: post
excerpt: "Working in multiple time zones can be difficult. I required a utility that would allow me at a glance to see the current date and time of a zone. Multizone is a simple bash script to fulfil this requirement."
---

Working in incident response its important to know exactly when an event occurred. It can quickly become confusing when working across multiple time zones, sometimes you just need to know when a country's work day starts. Even worse is dealing with timestamps without time zones specified, but then don't forget about Daylight Savings Time (DST) as well! 

I needed a simple date/time utility to serve as a quick reference. Thus [Multizone](https://github.com/passbe/multizone), a bash script which outputs the date, time and offset from a list of predetermined zones. The core of the script is just a simple loop: 

``` bash
for t in "${!timezones[@]}"; do 
    date=$(TZ=/usr/share/zoneinfo/"${timezones[$t]}" date +"$date_format")
    printf "%s|%s|%s\n" "${t}" "${date}" "${timezones[$t]}"
    sleep 0.1
done | sort | column -t -s '|'
```

You may notice there is no way to refresh the output. Why introduce code complexity when we already have the [watch](https://linux.die.net/man/1/watch) command:

``` bash
watch -t -n 30 multizone.sh
```

Combine this with [tmux](https://tmux.github.io/) and pane splits, and you have a utility to quickly glance at next time you need time zone information:

![Multizone in action](/assets/2017/multizone.png "Multizone in action")

You can clone the [github repo here](https://github.com/passbe/multizone).

- - -

Another titbit, the `date` command is incredibly powerful, you can convert time to any time zone you specify:

``` bash
date --date='TZ="America/Los_Angeles" 13:24 next Mon'
```

Just make sure you don't mistype the TZ otherwise you will experience inconsistencies. You can find out more here [https://www.gnu.org/software/coreutils/manual/html_node/Date-input-formats.html](https://www.gnu.org/software/coreutils/manual/html_node/Date-input-formats.html).

Now if only every date/time based system specified the time zone.
