---
title: Custom Scroll Restoration Proposal - Event-based API
layout: markdown
---

Custom Scroll Restoration - Event-based API
===============================================

## Scope

This document describes an API that gives web authors additional control over
user agent's restoration of persisted use state in particular the scroll
position and page scale.

## Design 

We propose to introduce a new cancelable event 'restorestate' (maybe
'restore'?).  This event will be dispatched before the user agent restores
various persisted user state and its default action is the restoration of
these states. Currently we propose only to give control over the restoration
of  scroll position and page scale. (In the future other persisted user state may
also be covered as well)

### Child frames

History traversal may result in the persisted user state of the nested frames
to also be restored. In such cases each frame receives a separate
'restorestate' event.

## Usage

Web applications that are interested in implementing their own app-specific
state restoration can listen to this event and prevent its default action. The
provided scroll position and scale value may also be used when implementing
their own restoration.

```js
window.addEventListener('restorestate', (event) => {
  // application restores scroll position as necessary
  // and prevents restoration of both scroll and scale
  event.preventDefault();
});

window.addEventListener('restorestate', (event) => {
  // Only use this if disabling scale independent of scroll is necessary
  event.preventScaleRestoration();

}); 
```

Checking for existence of the event type can be used as a way to feature
detect:

```js
let canPreventScrollRestoration = !!window.RestoreStateEvent
```

## Spec Changes

Replace step #9 in [history traversal algorithm][spec] with the following:

9\. If the specified entry has persisted user state, and the user agent wants
to update aspects of the document and its rendering it should run the
`persisted user state restoration` steps.

Add a new section that defines user state restoration

### Persisted user state restoration

Run the following steps immediately:

  1. Fire a trusted event with the name `restorestate` at the Window object of
  the Document, using the RestoreStateEvent interface, with scrollPostion and
  scale values initialised to the equivalent value from persisted user state.
  This event must bubble and be cancelable. Its default action is the
  restoration of scroll and scale from persisted user state.

  2. If the event default action is prevented do not execute the rest of steps

  3. If `preventScaleRestoation` is not called on the event restore the scale
  for the document.

  4. If `preventScrollRestoation` is not called on the event restore the
  scroll position for the document.

User agent may now update other aspects of the document and its rendering, for
instance values of form fields, that it had previously recorded.

### Web IDL


```webidl
[Constructor(DOMString type, optional RestoreStateEventInit eventInitDict), Exposed=(Window,Worker)]
interface RestoreStateEvent : Event {
  ScrollPosition? scrollPosition;
  unrestricted double? scale;

  void preventScrollRestoration();
  void preventScaleRestoration();
};

dictionary RestoreStateEventInit : EventInit {
  ScrollPosition scrollPosition;
  unrestricted double scale;
};

// Maybe use ScrollToOptions
dictionary ScrollPosition { 
    unrestricted double left;
    unrestricted double top;
};
```

[spec]: https://html.spec.whatwg.org/multipage/browsers.html#history-traversal