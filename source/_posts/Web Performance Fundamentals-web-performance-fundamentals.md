---
title: Web Performance Fundamentals
date:  2021-10-30 12:00:00
tags:
- performance
---
## Web Vitals
* First Contentful Paint (FCP) - RESPONSE QUICK
* Largest Contentful Paint (LCP) - GET TO THE POINT
* Cumulative Layout Shift (CLS) - DON'T MOVE STUFF
* First Contentful Paint (FID) - DON'T LOAD TOO MUCH

![web-vitals](/img/2021/10/Screenshot%202021-10-30%20at%2017.18.51-6364ed8f8a3143679e31e1d04976713b.png)

Google Chrome web vitals: https://github.com/GoogleChrome/web-vitals

## Lighthouse
Lab data vs Field data
https://www.lightest.app/

## Performance Analytics

* CLS is inversely correlated to Session Time
* LCP is inversely correlated to  Bounce Rate



## Optimizing Metrics

### Improving FCP

* Sized Correctly (Hardware, VM, network, etc)
* Minimal Processing
* Network Bandwidth (compression, cdn...)

### Improving LCP
> Defer resources until later

![LCP.png](/img/2021/10/LCP-135cf01d0ad2430d8189b54c7fa60d1d.png)

* async vs defer

>defer will defer js execution after LCP

![asyncdefer.png](/img/2021/10/async-defer-20b15158ae774abf8aa8493e286c6ccd.png)


* Lazy loading

We could use IntersectionObserver to detect when an element becomes visible in window and load it.

```
var lazyObserver = new IntersectionObserver(function(entries) {
      entries.forEach(function(entry) {
        if (entry.isIntersecting) {
          var el = entry.target;
          load(el);
          lazyObserver.unobserve(el);
        }
      });
    });

function load(el) {
    var src = el.getAttribute("data-src");
    var srcset = el.getAttribute("data-srcset");
    if (src) { el.setAttribute("src", src); }
    if (srcset) { el.setAttribute("srcset", srcset); }
    el.removeAttribute("data-src");
    el.removeAttribute("data-srcset");
  }
```

### Responsive Images

Let image downloads based on the viewport size.

![responsiveimages.png](/img/2021/11/responsive-images-2b48bdc9226e45c49af4b750b2e27587.png)


### HTTP2, Caching, and Pre-loading 

1. HTTP2 allows us to reuse the connections, yet not all servers support them
2. Caching
```
cache-control: max-age=600
expires: Wed, 20 Jan 2021 03:13:21 GMT
etag: "600a4af-c96f"
```
3. Preloading
Use annotations of preconnect and preload to preload CSS fonts => reduce the time between FCP and LCP


### Improving CLS practice
Be careful with relative position and prefer reserve space for large image

