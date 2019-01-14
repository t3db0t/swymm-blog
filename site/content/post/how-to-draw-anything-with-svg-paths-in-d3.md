---
title: How to Draw Anything With SVG Paths in D3
date: '2019-01-13T22:51:22-05:00'
---
D3 is great at many things, but overall the documentation ranges from “ok” to “unbelievably confusing or barely documented at all”.

Figuring out how to do a simple SVG path ended up being the latter.

But it wasn’t for lack of examples and tutorials—a google search turns up plenty, but not ONE of them did the one simple thing I was looking for: drawing my own SVG path from scratch. Just the [basic, core W3-defined paths](https://www.w3.org/TR/SVG/paths.html), without using the [path _generators_](https://www.dashingd3js.com/svg-paths-and-d3js) that almost all the tutorials refer to. Those are intended specifically to make it fast and easy to draw basic types of charts—line, bar, radial, etc.. That’s great, but not what I needed.

Now, the first obvious google result is the [D3 Path library](https://github.com/d3/d3-path), and that is indeed what we will use. But unless I’m somehow missing something, there is simply no useful example or tutorial on drawing a path using this API—which seems crazy or impossible. Nonetheless, here’s the missing tutorial!

Thankfully it turns out to be super simple, just unclear from the (very minimal) documentation in the library itself. Here’s an example:

```
var width = 300;
var height = 20;

var svg = d3.select("#d3")
 .append("svg")
 .attr("width", width)
 .attr("height", height);

 var path = svg.append("path");

 var pathData = d3.path();
 // Upper Right point
 pathData.moveTo(width, 0);
 pathData.bezierCurveTo(width, height, width-height, height, width-height, height);
 // Lower Left point
 pathData.lineTo(0, height);
 pathData.bezierCurveTo(0, 0, height, 0, height, 0);
 // Back to Upper Right
 pathData.lineTo(width, 0);
 pathData.closePath();

 path
 .attr("d", pathData)
 .attr("fill", "black");
```
Boom: here’s a bar with two bezier curves on it. Now that it’s parameterized, you can map those values to whatever you want.

This particular example is not the normal D3 style, in which data is bound to a selection using .enter() and so forth. For that, you need to use the generator methods (though I think there might be a way to do it with this API as well, I have to poke around some more).
