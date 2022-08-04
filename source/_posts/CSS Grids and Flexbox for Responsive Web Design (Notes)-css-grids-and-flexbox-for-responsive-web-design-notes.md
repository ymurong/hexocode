---
layout: post
author:  "Yanchao"
date:  2021-09-28 12:00:00
title: CSS Grids and Flexbox for Responsive Web Design (Notes)
tags:
	- css
---
This is my course notes which mainly summarised the best practices in web application layout.

Course Github link: https://github.com/jen4web/fem-layout

## Responsive Design Definition

1. Flexible grid-based layout
2. Media queries (CSS3)
3. Images that resize

## Float-based Grid System
> if you float, you must clear !

```
  .row::after {
      /* clear float formula */
      content: "";
      display: table;
      clear: both;
  }
```

Example: [Float-based Grid System Example](https://github.com/ymurong/css-exercise/blob/master/1-float/end/css/floats.css)

### Disadvantages
* equal height problem
* hard to do reordering 
* impossible to do spacing




## Flexbox-based Grid System

### Flexbox Introduction

1. The first layout elements - but not designed to lay out whole web pages
2. Features **flex-containers**(row) and **flex-items**(cells). Both are required to work together.
3. Excels at vertical centering and no equal heights problem
4. very easy to reorder boxes
5. Major disadvantages:
	* wasn't designed to be locked down for layouts! Works in 1 dimension only (one continuous row, it will wrap on to the next line as it runs out of screen space indefinitely)
	* browser support and syntax is challenging



### Flexbox Properties Cheatsheet

```
Flexbox Properties

Parent (Flex Container)
	display: flex | inline-flex;

	flex-direction: row | row-reverse | column | column-reverse;

	flex-wrap: wrap | nowrap | wrap-reverse;

	flex-flow (shorthand for flex-direction and flex-wrap)

	justify-content (main axis): flex-start | flex-end | center | space-between | space-around | space-evenly;

	align-items (cross axis - adjust to individual sizes): flex-start | flex-end | center | baseline | stretch;

	align-content (cross axis - adjust to largest item): flex-start | flex-end | center | stretch | space-between | space-around;


Children (Flex Items)
	order: <integer>;

	flex-grow: <number>; 

	flex-shrink: <number>; 

        // never use width on flex item, use flex-basis instead
	flex-basis: <length> | auto;

	flex: shorthand for grow, shrink, and basis (default:  0 1 auto)
	
	align-self: overrides alignment set on parent
```

Example: 
* [Flex-based Grid System Example 1](https://github.com/ymurong/css-exercise/blob/master/2-flexbox/end/css/flexbox.css)
* [Flex-based Grid System Example 2](https://github.com/ymurong/css-exercise/blob/master/4-flexbox-image-galleries/end/gallery-2.css)


### Responsive Images

Images shoud change size, based on screen resolution
* Load a big image and let it scale (not good)
* Server-side (good)
* Client-side: load several images and display the one right for this resolution (not good)
* Client-side: let js decide (better)

Better Solution: New picture tag released in HTML 5.1 with Picturefill polyfill to ensure backwards compatibiltiy

```html
<picture>
	<source srcset="img/peace-pie-original.jpg" media="(min-width: 1200px)">
	<source srcset="img/peace-pie-500.jpg" media="(min-width: 800px)">
	<img src="img/peace-pie-150.jpg" alt="My amazing peace pie at the appropriate dimension!">
</picture>
```

[Picture fill Link](http://scottjehl.github.io/picturefill/)

```
<head>
  <script>
    // Picture element HTML5 shiv
    document.createElement( "picture" );
  </script>
  <script src="picturefill.js" async></script>
</head>
```

#### srcset and sizes

the browser decides which image to download


#### Image downloads and performance
[Media Query & Asset Downloading Results](https://timkadlec.com/2012/04/media-query-asset-downloading-results/)
> using background in media queries could be a good practice and reduce unnecessary downloads

### CSS Grid (TODO)




### Some useful CSS tricks

1. [attribute selector documentation](https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors)
The CSS attribute selector matches elements based on the presence or value of a given attribute.

2. border-box model [box-sizing-border-box-ftw](https://www.paulirish.com/2012/box-sizing-border-box-ftw/)

```css
/* apply a natural box layout model to all elements, but allowing components to change */
html {
  box-sizing: border-box;
}
*, *:before, *:after {
  box-sizing: inherit;
}
```

### Some useful websites

1. https://caniuse.com/flexbox
2. http://flexboxgrid.com/