---
title: "cryptobin"
categories: crypto ctf
---

Over the holidays I spent some time on two different but related sets of challenges.

The first was the krypton wargame at [OverTheWire](https://overthewire.org). This wargame goes over some basic classical cryptography. I was a bit disappointed in the last level, I don't think the way I solved it was the intention of the author. It certainly didn't match the level text.

The second was the [AIVD kerstpuzzel](https://www.aivd.nl/onderwerpen/informatiebeveiliging/het-nationaal-bureau-voor-verbindingsbeveiliging-nbv/aivd-kerstpuzzel-2017), which is an annual Christmas puzzle produced by the Dutch equivalent of the CIA. This year's edition was particularly difficult (at least, this is what I read online). There was some cryptography, word puzzles, number sequences, and a lot of pattern recognition. I didn't get very far.

I want to do better on next year's puzzle. A lot of the problems seemed ripe for automation. Certainly there was a lot of general functionality needed to implement solvers for specific problems. After I gave up on the puzzle, I decided to implement my solutions to krypton in a useful way. It was also a good excuse to become more familiar with python.

Enter [cryptobin](https://github.com/avanpo/cryptobin).

Right now it's just a loose collection of utilities. I have two desires for this project. One is to provide command line utilities for common classes of problems. The other is to provide a large collection of useful python modules, in order to quickly hack together solutions for more specific cases.

I see several categories of tools:

* classical cipher utilities, including plaintext analysis for automation
* advanced dictionary search utilities
* number sequence tools (this one is more difficult, but at the very least a tool that pulls in all of the relevant information for each number)
* whatever else I can automate

Since I keep coming back to wargames and puzzles, this is one project I expect I'll be able to maintain.
