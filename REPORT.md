# WebKit bug report draft

**Title:** position:fixed descendant of a z-indexed element is clipped to body's box when html and body have overflow:hidden (hit-testing unaffected)

## Test case

Attach `minimal.html` and `minimal-body-height-548px.html` (the same page with `body { height: 548px; }`):

```html
<!DOCTYPE html>
<style>
  html, body { margin: 0; overflow: hidden; }
  #wrapper { position: relative; z-index: 2; }
  #banner { position: fixed; left: 0; right: 0; bottom: 0; height: 200px; background: red; }
  #cover { position: fixed; inset: 0; z-index: 1; background: white; }
</style>
<div id="wrapper"><div id="banner" onclick="this.style.background = 'green'"></div></div>
<div id="cover"></div>
```

## Steps to reproduce

Open the attached test case.

## Expected

A red banner at the bottom of the viewport. `#banner`'s containing block is the viewport, so `body`'s overflow clip must not apply to it. `#wrapper` (z-index 2) sits above `#cover` (z-index 1).

## Actual

The page is entirely white, but clicking the bottom 200px hits `#banner`, which turns it green, still invisibly. Hiding `#cover` reveals the banner. On iOS only a strip under the toolbar shows red.

`document.elementFromPoint()` at the bottom of the viewport returns `#banner`, yet viewport screenshots (including WebDriver's) are entirely white.

The banner is clipped to `body`'s box. Because `html` also has `overflow: hidden`, `body`'s overflow is not propagated to the viewport, so `body` clips its own box, which here is 0px tall. With `minimal-body-height-548px.html` in a 648px-tall viewport, the banner spans 448–648px and only 448–548px is painted. With `body { height: 300px; }` it stays invisible.

## It renders correctly when

- `overflow: hidden` is removed from either `html` or `body`
- `#cover` is `position: absolute`
- `#wrapper` has no z-index
- `#wrapper` is `position: fixed` instead of `relative`
- `body` is at least as tall as the viewport (`body { height: 100vh; }`)

## Reproduced in

- Safari 27.0 (21625.1.29.18.28) on macOS 26.7.1
- iOS 27.0 Safari (Simulator)
- Safari Technology Preview 27.0 (21626.1.8.19.2) on macOS 26.7.1

## Renders correctly in

- Chrome 152.0.7977.130
- Firefox 135.0 (headless)

## Impact in the wild

Cookie-consent banners such as Axeptio become invisible but still capture every tap on pages that lock scrolling under a fixed full-screen iframe. Users perceive this as a frozen page.

## Related

- Bug 212419 (RESOLVED FIXED, r262237): incorrect clipping of fixed elements inside a composited overflow:hidden stacking context. This case still reproduces in Safari 27. Here the clipping box is `body`, whose overflow is not moved up to the viewport because `html` also has `overflow: hidden`.
- Bug 160953: the same family of bug, where the z-indexed parent itself has the overflow.
- Bug 299084: possibly related. It uses the same html/body `overflow: hidden` scroll-lock pattern, and body/html overflow handling changed in Safari 26.
