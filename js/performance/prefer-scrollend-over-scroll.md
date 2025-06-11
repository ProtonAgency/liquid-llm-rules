# Prefer scrollend over scroll for performance where supported

Modern versions of Chromium and Firefox support the scrollend event (introduced in 2023), which is a more efficient alternative to the traditional scroll event.

This rule recommends using scrollend where available, with a fallback to scroll for unsupported browsers (e.g., Safari). This improves performance by reducing unnecessary firing of handlers during continuous scrolling.

Example Implementation:

```js
/*
  Chromium + FF introduced `scrollend` in 2023 for better performance
  Not yet available in Safari though 😬
*/
const scrollEvent = 'onscrollend' in window ? 'scrollend' : 'scroll';
document.addEventListener(scrollEvent, function () {
  // do something on scroll(end)
});
```
