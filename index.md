---
title: Custom Scroll Restoration Proposal
layout: markdown
---


Custom Scroll Restoration Proposal
==================================

Proposal to gives web authors additional control over user agent's restoration
of persisted use state in particular the scroll position and page scale.

There are two slightly different proposed API that can achieve this:

- [History based API](history-based-api.html): This is the original proposed API
   which is currently implemented in chromium behind [experimental feature
   flag][chromeflag]. ([whatwg discussion][whatwg])

- [~~Event based API~~](event-based-api.html): This proposal suffers from serious
  problem described in [issue 3](https://github.com/majido/scroll-
  restoration-proposal/issues/3).


## Problem Summary

History [spec][spec] does not specify the scroll position persistence and
restoration and leaves it up to user agents to implement. Despite this all major
browsers provide scroll restoration functionality for both traditional document
navigations and entries create via History API (i.e., pushState, and
replaceState). Despite its prevalence there is currently no reliable and cross-
browser solution for developers to opt out of this behaviour or control any of
its aspects.

The default scroll restoring behaviour works well for document style web sites
but it is often not appropriate for single-page web applications. Precise
controlling of the scroll restoration is particularly important for single- page
applications due to the following reasons:

1. They may be re-constructing the page content onpopstate every time and
often asynchronously. The user agent does not know when page is fully
constructed so it may attempt to restore scroll position on a partially
constructed page which has an incorrect size causing the scroll to land on an
unintended incorrect position due to clamping.

2. They may want to provide consistent scroll restoration regardless of how
the page is entered e.g., via history navigation or vanilla links clicks.

3. They may want to control the details of visual transition between UI states
for example using a specific scrolling animation to transition instead of an
instant jump, or even restoring scroll position of inner scrollers.

There several hacks used in practice which suffer from serious issues and force
developers to make unnecessary compromises. You can find a discussion of these
workarounds in appendix A. Few examples for developers struggling with these
issues are documented [here][issue1], [here][issue2], [here][issue3], and
[here][issue4].


## Existing Workarounds

There are several imperfect ways to work around the lack in the API. These are
brittle and have serious drawbacks. Two of the more popular ones are:

* Avoid document scrolling altogether by fixing body size and putting
content in an inner scrollable div. This is for example the approach
currently used in G+, and CBC news. Its drawback is that it requires a
specific page layout which breaks features that depend on document
scrolling such as top controls hiding and overscroll visuals. It also
relies on the assumption that the browsers will only persist document
scroll position which is not guaranteed.

* Override browser automated restoration by doing a second deliberate scroll
after it. For example this is the approach taken by Facebook, medium. This
can result in bad UX by causing two visible jumps.

As a side not both workarounds require rolling out a custom scroll state
persistence and restoration solution which needs to keep track of scroll
positions in order to restore them in future. Tracking document scroll position
is complicated and brittle because it is not possible to distinguish between
browser and user initiated scrolls and browsers have different scroll
restoration timing.  See [this bug][position-tracking-bug] for an example of
this brittleness that broke Photo Search.  Being able to disable browser scroll
restoration has the side benefit of making it possible to  have a simple and
reliable solution for tracking document scroll position (i.e., record scroll
position *onpopstate)*.


For some more details around background and motivation for this proposal 
see [here][background].

[background]: https://docs.google.com/document/d/1Tiu8PjvBtNOAgeh6yrs7bOrXxQcavQLiNtRJ_ToLlVM/edit
[spec]: http://www.w3.org/TR/html51/browsers.html#history
[whatwg]: https://lists.w3.org/Archives/Public/public-whatwg-archive/2015Mar/0070.html
[issue1]: https://bugzilla.mozilla.org/show_bug.cgi?id=679458
[issue2]: https://github.com/rackt/react-router/issues/707
[issue3]: http://andrz.me/blog/scrollx-scroll-why-history
[issue4]: https://aerotwist.com/blog/some-gotchas-that-got-me/#body-scrolling-is-impossible-to-stop

[position-tracking-bug]: https://code.google.com/p/chromium/issues/detail?id=474579
[chromeflag]: chrome://flags/#enable-experimental-web-platform-features
