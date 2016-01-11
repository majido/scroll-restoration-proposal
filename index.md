---
title: Custom Scroll Restoration Proposal
layout: markdown
---


*NOTE*: The proposed API is now part of [WhatWG HTML Living Standard][whatwg-spec]

Custom Scroll Restoration Proposal
==================================

This is a proposal for a [small new API](history-based-api.html)\* that gives
web authors additional control over user agent's restoration of persisted use
state in particular the scroll position. ([whatwg discussion][whatwg])

\* This is implemented in chromium and it is [planed][chromestatus]
for release in version 46.


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
issues are documented for [Mozilla][issue1], [Facebook][issue2], [React router][issue3],
[Google products][issue4], [Mobile SPA][issue5], and [others][issue6].


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


## Alternative APIs Considered

1. Use [events][2] to allow detection of scroll restoration and its
prevention. This [forces the timing of restoration][3] to be after DOMReady on
cross-document navigation. This is an unacceptable UX.
2. Use a [fourth additional options on pushState, replaceState][4]. Deemed too
complex for simple scenarios and [unnecessarily links state creation with
scroll restoration][5].
3. Same as #2 but with [two brand new methods][6] (push, replace) that accept
dictionaries.

For even more minor API variation considered see [this document][background].



[background]: https://docs.google.com/document/d/1Tiu8PjvBtNOAgeh6yrs7bOrXxQcavQLiNtRJ_ToLlVM/edit
[spec]: http://www.w3.org/TR/html51/browsers.html#history
[whatwg]: https://lists.w3.org/Archives/Public/public-whatwg-archive/2015Mar/0070.html
[whatwg-spec]:https://html.spec.whatwg.org/multipage/browsers.html#dom-history-scroll-restoration
[issue1]: https://bugzilla.mozilla.org/show_bug.cgi?id=679458
[issue2]: https://bugs.webkit.org/show_bug.cgi?id=51899
[issue3]: https://github.com/rackt/react-router/issues/707
[issue4]: https://crbug.com/444094
[issue5]: http://andrz.me/blog/scrollx-scroll-why-history
[issue6]: https://aerotwist.com/blog/some-gotchas-that-got-me/#body-scrolling-is-impossible-to-stop

[position-tracking-bug]: https://code.google.com/p/chromium/issues/detail?id=474579
[chromeflag]: chrome://flags/#enable-experimental-web-platform-features
[chromestatus]: https://www.chromestatus.com/feature/5657284784947200

[2]: http://majido.github.io/scroll-restoration-proposal/event-based-api.html
[3]: https://github.com/majido/scroll-restoration-proposal/issues/3
[4]: https://lists.w3.org/Archives/Public/public-whatwg-archive/2015Mar/0070.html
[5]: https://lists.w3.org/Archives/Public/public-whatwg-archive/2015Jul/0052.html
[6]: https://lists.w3.org/Archives/Public/public-whatwg-archive/2015Mar/0166.html
