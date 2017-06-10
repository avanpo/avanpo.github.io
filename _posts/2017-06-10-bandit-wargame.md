---
title: "Bandit wargame"
categories: ctf
---

Today I finished the bandit wargame at [OverTheWire](https://overthewire.org). It was a lot of fun! It's aimed at players looking to become familiar with the command line and basic linux tooling.

I've only recently begun to appreciate how many useful tools are contained in coreutils. I got to use a lot of utilities that I didn't even know existed. A simple wargame like this is an excellent way to learn them.

I was familiar with nearly all of the concepts touched on in the levels. Despite this, I still managed to get stuck several times. Understanding the concepts on a higher level often isn't enough -- sometimes you really need to understand the utility intimately to get the job done. Either that or try everything you can think of until it works. Usually I avoid doing that out of principle, which means I need to read man pages a lot.

Some of the quirks that cost me a lot of time:

* While `setuid` works as advertised for binaries, it does not work for shell scripts. This is due to the way that the OS executes shell scripts. While it could easily be fixed, it turns out that there are some very compelling reasons to leave things as they are. See this [question and answer](http://www.faqs.org/faqs/unix-faq/faq/part4/section-7.html).
* The wargame was migrated to a docker setup after I started, meaning each ssh session connects to a fresh container. It took me a while to realize that this was why I wasn't able to connect to a port I was listening on (from another container).
* I ran into a weird bug of some sorts where generating a rather large number of lines and piping them to netcat terminated the connection prematurely. However, redirecting the generated lines to a temp file, and then redirecting that temp file to netcat worked. I haven't figured that one out yet.
* You can use vim to run arbitrary shell commands. But if your shell is not a typical shell, then the path may not be set and no commands will be available. Luckily vim can navigate the file system itself, and even set and spawn a specific shell.

All in all I learned a lot of useful things. Highly recommended as an introduction to wargames, or to patch up knowledge on commonly used linux utils.
