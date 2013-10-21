---
layout: post
title: Using SVG to lay out HTML5 elements
author: joel
tagline: Introspection via JS to extract position information.
category: technical
tags: [svg, html5]
image:
  feature: texture-feature-03.jpg
---

The problem: Take an SVG, and use the location of specific SVG elements to display
HTML5 content.

TODO:

- How much background to give re: using SVG for form generation?
- Example images and code examples would make this more useful, but require more work.

## The first hopeful solution

At first, my research pointed me to a promising SVG element called
`foreignObject`. This object is specifically designed to allow for the
embedding of XML content within SVG structures. My general impression from
some googling was that both HTML and MathML foreignObject content are
supported by browsers.

"Excellent!" I thought.

Some simple HTML was all I needed, and I was
temporarily glad I would not have to muck around with transforming
coordinates manually.  I could leave it up to the browser to put things in the
right place. Or at least, so I thought...

I was using `<input>` elements, and these worked okay... so long as there were
[no transforms involved](http://code.google.com/p/chromium/issues/detail?id=116566).
This unfortunately makes this approach unusable for all but the simplest SVGs.

I looked for alternatives to getting an input-like element working. On
StackOverflow I found someone attempting a [corrected rotated editable text
area](http://stackoverflow.com/questions/12759527/multiline-editable-textarea-in-svg).
This was similar to my use-case of soliciting information from users.

I tried the `content-editable` approach, but quickly got frustrated with long-standing
[Chrome/Webkit bugs](https://bugs.webkit.org/show_bug.cgi?id=71819). A comment
on that bug indicates _"There are numerous Chromium bugs surrounding transforms
of foreign objects and events."_. This encouraged me to abandon this approach
and try something else.

This [video example](http://double.co.nz/video_test/video.svg) shows both the
potential promise of HTML foreignObjects in SVG, but also highlights the
problems between browsers. While viewing the example, I noticed the following
problems in recent browsers:

- Firefox: Videos are in the right place and can be moved around. But the rectangles can't be resized.
- Chrome: Videos in wrong place, but rectangles can be moved around, resized, rotated.

## The pragmatic workaround

Having resigned myself to SVG `foreignObject` being somewhat sketchy, and
needing a solution in a shorter time than patching Chromium would entail, I started looking at how I could
manually position elements on top of an SVG.

On the d3 news group, [Ian Johnson](http://enja.org) posted this snippet of
javascript for getting the position of an SVG element:

~~~ javascript
var p = element.nearestViewportElement.createSVGPoint();
var matrix = element.getTransformToElement(element.nearestViewportElement);
p.x = 0; //bbox.x;
p.y = 0; //bbox.y;
var sp = p.matrixTransform(matrix);
~~~

This creates a new point _p_ (x,y) object in the SVG, and transforms it by the transformation matrix of the element we're interested in.

This looked promising but annoyingly, I ran into browser inconsistencies again.
Chrome supported `getTransformToElement` with a null parameter, and this usage
would align things perfectly. Firefox doesn't support this invocation,
requiring a SVGElement argument.  Using `nearestVieportElement` as the above
snippet did would leave things slightly askew.

Wanting more information on `getTransformToElement` I looked around. The
standard wasn't too enlightening, and I wasn't the only one looking for some
more details. [This answer on StackOverflow](http://stackoverflow.com/a/6084322/272238) provided insight.

### getCTM() to the Rescue

Still, despite usage of getTransformToElement, and using various elements and
arguments, I couldn't get consistent behaviour. Eventually I came across
`getCTM()` which works out the the correct transformation matrix for an
element, without needing extra parameters. It also worked consistently between
Chrome and Firefox.

### Remembering local coordinate offsets

Remember that the tranformation matrix is just manipulating the local
coordinate space. Thus, you still have to take into account any local
positioning. For rect elements this requires checking for the x and
y coordinates, or for circles, the cx, cy (center x, center y) coordinates. I'm
not sure about other valid SVG element positioning.

In addition, you have to take into consideration any other points of interest.
If you want a rectangle, then that might be the bottom-right corner in addition
to the top-left. You can't just add the height width to the transformed point,
because the transformation matrix can potentially warp the sizing of the
element. In other words, the width/height are in local units, but we're
after absolute points.

