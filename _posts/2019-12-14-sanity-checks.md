---
title: "Sanity checks"
categories: sre opinion
---

In my line of work I've dealt with a lot of automated data pipelines that feed
into production systems.

Validating the output of these pipelines in an automated manner is frequently
difficult. Suppose you had a different data source you could validate against.
If you could reliably tell whether your pipeline was producing invalid data
using this data source, why not directly incorporate this data into your
pipeline? Alternatively, if the data source isn't reliable enough to do so,
chances are you'll need a human to come look at the diffs either way.

Another option is to simulate the production system under the new data, using
historical or current load. This is often prohibitively expensive and a large
maintenance burden, and usually impossible for data that rolls out at a high
cadence (sometimes this is unavoidable). It can work well in select scenarios,
however. This assumes you have good monitoring in place with a high degree of
coverage.

If your production system tolerates data version skew between instances, you can
get around validation by simply canarying the data to a small fraction of your
production instances. While this should prevent global outages, depending on the
nature of the service it might allow frequent smaller outages that eat into your
SLO. Proper semantic canarying might also be infeasible for data that rolls out
at a high cadence, but it's still worth the effort to catch crashes and other
immediately apparent problems.

The simplest and least expensive way to validate pipeline output is by sanity
checking the data before rolling it out. In my experience, simple absolute
checks like making sure the output (or input) is not empty will go a long way in
preventing outages. For more complex pipelines, performing these checks at every
stage can be very effective.

Absolute sanity checks may quickly become dated if the data experiences natural
growth.  In this case, it is common to compare the new data against the data
currently in production (which is assumed to be valid). If the new data differs
from the old by some threshold, the new data is considered to be invalid and the
pipeline stalls. This works well when the data is difficult to validate at face
value, even by a human.

Usually it's difficult to determine what this threshold should be when you're
writing the check. I think most people just pick something that looks
reasonable, instead of doing a thorough analysis. I've heard people imply these
kinds of checks may as well be removed, given the arbitrariness of the
threshold. I've never really understood this argument. Often, it's really
difficult to do any kind of meaningful analysis. Data changes over time, and
it's likely that any analysis will quickly be dated.  Surely having a (loose)
relative check is better than not having one at all?

Relative sanity checks have the problem of being noisy. In my experience only a
small fraction of alerts based on relative checks expose actual problems. But
this small fraction also prevents serious outages. Over time, well-tuned
thresholds can reduce the amount of noise, but it's hard to make it go away
entirely. This noise in itself can cause outages; I've seen two main classes of
this, both caused by humans.

The first is in a poorly automated system, where engineers frequently need to
inspect failing relative checks. This tends to cause fatigue. A failing relative
check due to a serious issue might be hidden beneath many other failing checks,
or masked by the fact that the failure is usually benign. If the usual response
is to simply let the data through, it's hard to blame the engineer when this
inevitably causes an outage. This usually indicates a problem with the pipeline.
If a check fails often, either relax/remove it or fix the underlying issue.

The second is when a sanity check fails, and the engineer has correctly
determined that the failure is benign. The engineer reruns the pipeline with
sanity checks disabled, at which point a different bug or data problem manifests
itself and causes a serious outage when the data rolls out to production.

This is solved by making it easy for an engineer to rerun the pipeline with
selectively relaxed thresholds.  Risk can be further minimized by making sure
the pipeline is hermetic and deterministic. A properly hermetic pipeline also
makes it much easier to debug issues after a failure.

You could make a case for rolling out the output that failed the benign sanity
check rather than rerunning the pipeline, but this requires extra logic to hold
on to the data until a human is able to look at it and roll it out. If this code
path is not very well tested, it could also cause an outage. This is especially
the case if it is not executed very often. Additionally, if the pipeline has
been stalled for a while, a large enough delta may have accumulated that the
next run fails as well. In this case, being able to selectively relax thresholds
is crucial anyway, meaning the extra complexity required to push after failing a
check is usually not worth it.

Pipelines that are stalled too long are often problematic when they rely on
relative checks for validation. If the data is not easily sanity checked by a
human, you can be left completely blind to whether or not the data makes sense.
The longer the pipeline remains stalled the larger the delta and the worse the
problem becomes. The easiest way to deal with this is by making sure it doesn't
happen in the first place. Otherwise, having a good monitoring and canarying
story is often the only way out.
