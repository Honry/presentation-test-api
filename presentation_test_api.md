# Presentation API Testing API

This is a draft API intended to automate Web Platform Tests for the Presentation
API.  It is not intended to be used by Web developers who wish to test
applications that use the API, although it may prove useful in that aspect.

## Interface

All interfaces listed below SHOULD be implemented by a controlling user
agent.  The interfaces MAY only be exposed when the user agent is started with a
specific flag.

```
partial interface Presentation {
  [SameObject] readonly attribute PresentationTest test;
};

interface PresentationTest {
  void addScreen(FakePresentationScreenInit fakeScreen);
  Proimise<DOMString> startDefaultPresentation();
  void reset();
};

dictionary FakePresentationScreenInit {
  required DOMString name;
  required sequence<DOMString> urls;
  optional boolean selected = true;
};
```

## PresentationTest Interface

The behavior of the Presentation API MUST not be altered in any way until
`navigator.presentation.test` is accessed.  If the `navigator.presentation.test`
attribute is retrieved in a browsing context, then the following steps happen:

1. An instance of `PresentationTest` is created and assigned to that attribute.
2. The `set of presentation controllers` for that browsing context is set to the
   empty list.
3. The `list of available presentation displays` for that browsing context is
   set to the empty list.

The test object lives as long as the browsing context that accessed it does.

## Adding Fake Screens

To add a fake screen, the page should construct a `FakePresentationScreenInit`
instance and pass it to `addScreen()`.  The fake screen will be added to the
`list of available presentation displays.` Its list of `compatible presentation
URLs` for the screen will be set to `urls`.

## Selecting a Fake Screen

If a fake screen is added with `selected` set to `true,` that screen will become
the selected screen for the next request to start a presentation.  If multiple
fake screens are added with `selected` set to `true,` then the implementation
acts as if no screen has been selected (i.e., the start Promise will remain
unresolved).  If no fake screen has `selected` set to true, the user agent will
behave as if the user has denied permission to use any display.

## Starting a Default Presentation

The `startDefaultPresentation()` method runs the steps to start a presentation
from a default presentation request with the currently selected screen (if any).
If the request is successful, the Promise is resolved with the name of the fake
screen.  If the request is not successful (no default request or no selected
screen), the Promise is rejected.

## Resetting

`reset()` will remove all fake screens, terminate all presentation connections
created with fake screens, and empty the `set of presentation controllers` and
the `list of available presentation displays.`

## Changes to Presentation API steps

When a `PresentationTest` object exists for a browsing context, the following
algorithms from the Presentation API specification are modified as follows:

* *Selecting a presentation display* 
  * Step 1 is bypassed.
  * Step 8 behaves as if the user had selected the chosen the selected fake screen
  (if any).
  * If there is no selected fake screen, step 10 is run.
* *Starting a presentation from a default presentation request*
  * These steps are run when `startDefaultPresentation()` is called.
* *Monitor the list of available presentation displays*
  * In step 5, the list of available presentation displays is set to the list of
    fake screens (possibly an empty list).
* *Starting a presentation connection*
  * Step 12 must create a receiving browsing context that implements the
    `PresentationReceiver` API along with a presentation connection to that
    browsing context.
  * These steps are not related to the fake screen itself; the behavior is
    identical regardless of which fake screen was selected.

## Implementation of a Fake Screen

The implementation of the fake screen itself is up to the user agent.  To
simplify debugging of tests, a typical implementation would open a local browser
window that navigates to the presentation URL.

