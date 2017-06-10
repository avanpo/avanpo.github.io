---
title: "Bandit wargame"
categories: ctf
---

Today I finished the bandit wargame at [OverTheWire](https://overthewire.org). It was a lot of fun! It's aimed at players looking to become familiar with the command line and basic linux tooling. I got to use a lot of utilities from coreutils that I didn't even know existed.

I was familiar with nearly all of the concepts touched on in the levels. Despite this, I still managed to get stuck several times. Understanding the concepts on a higher level often isn't enough -- sometimes you really need to understand the utility intimately to get the job done. Reading man pages is good for you.

All in all I learned a lot of useful things. Highly recommended as an introduction to wargames, or to patch up knowledge on commonly used linux utils.

### Things I learned

* While `setuid` works as advertised for binaries, it does not work for shell scripts. This is due to the way that the OS executes shell scripts. While this could be fixed, it turns out that there are some very compelling reasons to leave things as they are. See this [question and answer](http://www.faqs.org/faqs/unix-faq/faq/part4/section-7.html).
* The wargame was migrated to a docker setup after I started, meaning each ssh session now connects to a fresh container. So if I set up netcat to listen on a certain port in one session, and try to connect from another, it obviously won't work. In my defense, the hint hadn't been updated to reflect the change in infrastructure (it has now).
* I ran into a weird bug of some sorts where generating a rather large number of lines and piping them to netcat terminated the connection prematurely. However, redirecting the generated lines to a temp file, and then redirecting that temp file to netcat worked. I haven't figured that one out yet.
* You can use vim to run arbitrary shell commands. But if your shell is not a proper shell at all, then the path may not be set and no commands will be available. This should also be obvious. Luckily, vim can navigate the file system itself. It can even set and spawn a specific shell for you.
