---
title: Growing Your Engineering Team? Embrace the Rewrite
description: Why rewriting parts of your stack is a good thing when growing an engineering team.
date: "2018-09-11T14:50:16.747Z"
---

[Dan Abramov](https://twitter.com/dan_abramov) recently had a good thread on
the resistance programmers have to touching production code that has been
working for awhile.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">There is a common wisdom that if some code works and has been in production for a long time, it’s better not to touch it. But I learned an interesting counterargument (I think from <a href="https://twitter.com/dmwlff?ref_src=twsrc%5Etfw">@dmwlff</a>?)</p>&mdash; Dan Abramov (@dan_abramov) <a href="https://twitter.com/dan_abramov/status/1039516417173770241?ref_src=twsrc%5Etfw">September 11, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

The thread is worth a read, but to summarize the argument, the longer a piece
of code hasn't been touched by anyone on your team, the greater the likelihood
that no one knows how it works. If the code in question is critical to your
product, you now have a combination of ignorance and fear.

This got me thinking about any time I've seen engineering teams in hiring mode.
After a few weeks of getting comfortable, the new individuals usually spot a
part of the code base that has been rotting for awhile and deem it prime for a
rewrite. The rewrite suggestion is usually coupled with ideas about how to
leverage the latest and greatest languages, frameworks and technology to solve
all the previous ills.

Product managers express frustration, "Rewrites always take too long and they
we won't ship anything new for months." While this is often true, I think this
is where Abramov's point is particularly important.

For a new team to be productive on a code base, they need to be able to make
rapid changes to it with confidence. Small, incremental changes to existing
systems can develop this confidence, but there are likely situations where a
the cost of a rewrite today is worth it to buy you the velocity you'll gain in
a few weeks time.

The balance that needs to be struck, is what is the minimal amount of changes
that can be done to achieve that confidence for the team in question. The
question to ask the team is "what are the minimal amount of changes you need
feel confident maintaining this code and building off of it?"

This framing feels more constructive for a conversation between product
managers and engineering teams than a flat out argument over the merits of a
rewrite in the first place.
