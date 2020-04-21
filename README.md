An Introduction to D3

Introduction
============

[D3](https://github.com/d3/d3/blob/master/API.md) is a popular Javacript
visualization library. There are tutorials online, but I have found that
they all assume intimate knowledge of the Javascript language and its
programming conventions. This tutorial covers the needed background
material, before presenting the D3 material

D3 is a surprisingly low level language given its popularity. With it,
one plots objects at specified XY locations on the screen. Every object
has to be specified. Despite this, D3 provides many primitives and
helper routines along with programming conventions that make it easy to
make complex visualizations with a small amount of code. If you want a
higher level language, take a look at
[Plotly](https://plotly.com/graphing-libraries/). It is written on top
of D3.

D3 is not an imperative programming language. Execution is driven by
data flow (that’s what the name stands for: Data Driven Documents),
which can be confusing to programmers familiar with imperative
languages.

When looking at D3 code, you need to be aware of which version is being
used. All of the examples in this document are done with D3 version 5.
However, the file attachments include both the version 3 and version 5
implementations for many of the visualizations

Javascript Conventions
======================

There a couple of Javascript programming conventions that you need to be
aware of before using D3. They are covered here.

Function Call Chaining
----------------------

The examples in this section come from [Understanding Method Chaining In
Javascript - Backticks & Tildes –
Medium](https://medium.com/backticks-tildes/understanding-method-chaining-in-javascript-647a9004bd4f).
Refer to it for a more detailed explanation.

Function chaining is a programming idiom that addresses the need to pass
the results of one function call to another. This happens quite a bit
when writing GUIs. Without chaining, code typically looks something like
this:

const myObject = new ClassIWrote()

myObject.someMethod(

myObject.someOtherMethod(

myObject.yetAnotherMethod()

)

)

While this example is pretty simple, it quickly gets ugly the more
complicated the code gets. Also, the control flow is unintuitive. The
flow is from the 4^th^ line and goes backward: yetAnotherMethod()→
someOtherMethod() → someMethod() . This is easily misunderstood.

Function call chaining is an alternative programming style that
simplifies things. It does this by introducing a convention. Each
function returns the object it is working on. This leads to code like
this:

myObject = new ClassIWrote().

.someMethod()

.someOtherMethod()

.yetAnotherMethod()

Not only is this easier to look at, the control flow is as expected:
someMethod() → someOtherMethod() → yetAnotherMethod() .

For this to work, each function has to return the object it is contained
in (stored in the variable **this**) or a new object that is
instantiated and has methods that behave similarly.

D3 heavily uses function chaining and you have to understand how it
works and be aware of one the teobject being operated on changes
(typically by creating a new object or set of objects).

Javascript Arrays
-----------------

In Javascript,
[arrays](https://www.w3schools.com/jsref/jsref_obj_array.asp) behave a
little differently than in other programming languages. Like everything
else in Javascript, arrays are objects. They have attributes and
functions (methods). Here are some unusal array methods that are useful
when writing D3 visualizations:

-   *array*.forEach(func, …) : apply a function to each member of the
    array
-   *array*.map( func , …) : return a new array, the result of a
    function called on each array member
-   *array*.filter(func, …) : filter the array by the user
    provided function.
-   *array*.sort(func) : sort the array by the user provided function.

Note that each one of these functions takes a callback function as the
first parameter. Passing functions to functions is commonly used in D3
and is quite powerful.

There’s one other array function I want to mention. Javascript arrays
only support integer indexing, so if you want to extract a piece of the
array you have to explictily call the slice function.

*array*.slice(*start,end*)

Indices start at 0 and the end parameter is optional. Negative values
are used to start from the end of an array.

Document Object Model (DOM)
---------------------------

D3 does not stand on its own, as it is built on top of Javascript’s
existing HTML Document Object Model (DOM). It is also designed to work
with Cascading Style Sheets (CSS), but that will not be covered here.

The objects D3 creates are all part of the HTML specification, so you
will want to keep an [HTML
Reference](https://www.w3schools.com/tags/default.asp) handy when using
them. For visualizations, D3 is used with graphics objects but it can be
used with any HTML element. And while all of the graphics objects D3
uses are part of HTML, only one of them, SVG (Scalable Vector Canvas),
is considered part of the HTML specification. SVG is the element that
defines the canvas to draw on (but is different than the HTML canvas
tag, which is a different kind of canvas used for making web games). The
other graphical elements are defined by the SVG XML specification. I
mention this so that you know where to look for the [SVG
Reference](https://www.w3schools.com/graphics/svg_reference.asp).

D3 Preliminaries
================

With the Javascript concepts covered, now we are ready to describe how
D3 builds upon it.

Selections: D3 and the DOM
--------------------------

A fundamental part of D3 is how it interacts to the DOM. It uses a
series of classes called
[*selections*](https://github.com/d3/d3/blob/master/API.md#selections-d3-selection),
where a group of objects are selected by some tag and operated on in
tandem. (jQuery also uses a similar selection concept). This is
motivated by the clunky syntax that basic Javascript provides for
styling objects. If you wanted to set the color of all the paragraphs in
javascript, here is how that is done:

*var paragraphs = document.getElementsByTagName("p");*

*for (var i = 0; i &lt; paragraphs.length; i++) {*

* var paragraph = paragraphs.item(i);*

* paragraph.style.setProperty("color", "blue", null);*

*}*

To do this in D3, this is done by:

*d3.selectAll("p").style("color", "blue");*

All of the paragraphs are selected, and then updated using the style
function. Note that this uses the function chaining idiom discussed
above. The selectAll() function creates a selection, and the style
function returns the same selection. This means that you can cascade
style calls together:

*d3.selectAll("p").style("color", "blue")*

*.style("fill", "blue");*

****

**Note that each style operates on the whole selection, returning the
selection. And then the second style operates on the set of results from
the first style. **D3’s **selections **classes have many methods that
operate similarly. However, some methods, such as **insert** creates new
elements. These methods still return a selection, only the selection is
different: the newly created elements. So it is common to see code like
this:**

*d3.selectAll("svg")*

*.style("background-color", "black");*

* .append("circle")*

* .attr("class", "dot")*

* .attr("r",3.5)*

* .append("title")*

* .text(“My title”) ;*

* *

**First the SVG canvas is selected. First the style of the canvas is
changed, setting its background color. Then a circle is appened, and its
attributes are set (class, and radius) . Then a title is appended to the
circle, and it’s text attribute is set.**

**Note that the first step was a select all on the SVG canvas. In the
common case, there is only one but there could be multiple. In that
case, each one of those SVGs would be operated on, theirs style changed
and the elements added to each. The end result is a selection containing
title elements, one for each SVG element.**

**At **the root of the document, there will typically be only one canvas
in your selection. But when working with data, the set of things
selected and what elements are being operated on becomes important.**

**The above examples all used the HTML element type as the selection
criteria. D3 uses the standard HTML/**CSS **
**[**select**](https://www.w3schools.com/css/css_selectors.asp)[**ors**](https://www.w3schools.com/css/css_selectors.asp)[**
API**](https://www.w3schools.com/css/css_selectors.asp)**. Which means
things can be selected by:**

-   **element: **Elements of the specified HTML type** E.g. “svg”**
-   **class: **Elements with the specified class attribute** E.g.
    “.**graph**” **
-   **id: **Elements with specified id attribute. Ids are unique to a
    page, so they will only selected one element** E.g. “\#id”**

**When inserting elements into a page, they are **often** given class
names or ids so that they can be selected later in the code.**

Structure of D3 Programs
------------------------

The data driven structure of D3 programs can be confusing to the
newcomer There are two main parts of D3 programs: setup and rendering.

In the setup section, variables are defined that are used in the
rendering section. These variables may look like objects being
instantiated, but they are not. The functions have attributes and
methods, so the look and smell like objects, but they behave
differently. As functions, their position in the code does not matter.
In Javascript, all the functions are defined on page load. So in many of
the online examples, these functions are defined after the main
rendering code. I’m not sure if this is done for performance reasons or
for stylistic ones, but you need to be aware of this when looking at
other people’s code. Some common types of D3 utility functions are
described below. There are many more.

### Scales

[Scales](https://github.com/d3/d3/blob/master/API.md#scales-d3-scale)
are another fundamental D3 building block. They are functions that
convert your data into the X or Y location, in pixels, on the screen.

Two scales have to be defined, one for the x direction and one for the y
direction. For each one you have to specify the size of the screen (the
range) and the limits of your data (the domain). The terminology comes
from algebraic functions. The domain describes the input, the range the
output. There is a method for specify each:

*scale*.range()

.domain()

There are different kinds of scales, depending on the type of data you
have. Numeric data uses
[continuous](https://github.com/d3/d3/blob/master/API.md#continuous-scales)
scales, such as:

-   scaleLinear : linear
-   scaleLog : log
-   scaleTime : dates and times

For categorical data,
[ordinal](https://github.com/d3/d3/blob/master/API.md#ordinal-scales)
scales are used:

-   scalePoint: maps to a point ; used in scatterplots
-   scaleBand : maps to a range ; used in bar charts
-   scaleOrdinal : Does not map to a position, but from a set of data
    values to a corresponding set of visual attributes (such as colors).

In earlier versions of D3, banding was an attribute of ordinal scales
and not a separate scale type.

There seems to be a convention where the x axis scale is stored in the
variable x and the y axis scale is stored in the variable y. E.g.

var x = d3.scaleLinear()

 .range(\[0, width\] );

var y = d3.scaleLog()

 .range(\[height, 0\]);

Again, these are defining functions that will be called when the graph
is rendered. Width and height are variables defined elsewhere in the
code. Their units are pixels.

When plotting to the screen, it is important to know where the origin
point ( x=0, y=0) is. It is the top left part of the screen. For the x
direction, increasing numbers go from left to right. For y, top to
bottom. So when you specify a range of \[ min, max\], you are saying
that you want the minimum value at pixel 0 and the maximal value at the
maximum pixel (width or height). That is fine for the x-axis, but not
for the y-axis. Typically, one wants the the (0,0) point to be the
bottom left in the chart. So to tell the scale that you want the minimum
data at the bottom of the screen, you specify a range of \[max,min\] .

### Axis

D3 has utilities for creating functions that [drawthe x and y
axis](https://github.com/d3/d3/blob/master/API.md#axes-d3-axis) on the
screen. The functions are called generators. Note that their only
function is to draw the axis line, tick marks, and associated labels.
There is no data bound to them.

There are two main parameters to specify for an axis, the scale and the
orientation. For the scale, you pass in one of the functions defined
above. The orientation is one of: top, bottom, left or right. The
orientation determines two things: the direction the line is drawn and
the where the tick labels are drawn relative to the axis. It does not
specify where the line is drawn. That can be controlled in the render
code. In D3 version 5, the orientation is specified by the constructor:

-   axisTop
-   axisBottom
-   axisLeft
-   axisRight

Here is an example:

var yAxis = d3.axisLeft()

 .scale(y) ;

var xAxis = d3.axisBottom()

 .scale(x) ;

Again, xAxis and yAxis are functions that will be called later, when the
graph is rendered.

Data binding
------------

There are two types of objects D3 programs have to deal with. The
graphical DOM elements and the data to be visualized. When visualizing,
the two types of objects are bound together. Data is bound to a
*selection *using the
[*data*](https://github.com/d3/d3-selection/blob/v1.4.1/README.md#selection_data)[*()*](https://github.com/d3/d3-selection/blob/v1.4.1/README.md#selection_data)[
](https://github.com/d3/d3-selection/blob/v1.4.1/README.md#selection_data)method.
For more control, the
[*join*](https://github.com/d3/d3-selection/blob/v1.4.1/README.md#selection_join)[*()*](https://github.com/d3/d3-selection/blob/v1.4.1/README.md#selection_join)[
](https://github.com/d3/d3-selection/blob/v1.4.1/README.md#selection_join)method
can also be used.

The *data* method returns a new selection, but it is a special type of
selection. Fundamentally, D3 is concerned with visualizing data
dynamically. As new data comes in, the visualization is updated. So when
data is bound to a selection of elements, there are three scenarios:

-   elements with data already bound to them that has been updated
-   data with no elements it is bound to
-   elements that no longer have data associated with them

The selection returned by *data()* is the preexisting elements that
already had data and need to be updated. It also contains to
sub-selections, one for the new data and one of the elements with no
data. The new data selection is accessed with the *enter()* method. The
empty elements are accessed with the *exit()* method. D3 code normally
has three sections, to handle all three cases. E.g.

*// Update existing*

*var p = d3.select("body")*

* .selectAll("p")*

* .data(\[4, 8, 15, 16, 23, 42\])*

* .text(function(d) { return d; });*

*// Data needing new elements*

*p.enter().append("p")*

* .text(function(d) { return d; });*

*// Old elements with no data*

*p.exit().remove();*

The *text()* method sets the text value for all elements in the
selection. While this example has all three sections, code using static
data will often have only the enter or update section. In the latter
case, all the elements have to be defined before binding.

Your First D3 Visualization
===========================

With all the preliminaries out of the way, we can now do our first D3
visualization.

Scatterplot
-----------

The starting point is a empty HTML document that loads D3. (Some online
tutorials take a different approach, mixing D3 code with HTML markup.)

&lt;!DOCTYPE html&gt;

&lt;html&gt;

&lt;head&gt;

 &lt;meta charset="utf-8"&gt;

 &lt;title&gt;Scatter plot D3 Version 5&lt;/title&gt;

 &lt;script src="https://d3js.org/d3.v5.min.js"&gt;&lt;/script&gt;

&lt;/head&gt;

&lt;body&gt;

&lt;/body&gt;

&lt;/html&gt;

Next, we setup variables and the utility functions. We are defining a
margin so that there is space to view the axis labels.

var margin = {

 top: 20,

 right: 20,

 bottom: 30,

 left: 50

},

 width = 600 - margin.left - margin.right,

 height = 300 - margin.top - margin.bottom;

var x = d3.scalePoint()

 .range(\[0, width\] );

var y = d3.scaleLinear()

 .range(\[height, 0\]);

var yAxis = d3.axisLeft()

 .scale(y) ;

var xAxis = d3.axisBottom()

 .scale(x) ;

var color = d3.scaleOrdinal(d3.schemeCategory10);

Next we instantiate the SVG canvas. The dimensions are set, and a
subgroup is declared that is shifted by the margins. It is in this
subgroup where the data will be rendered.

var svg = d3.select("body").append(o"svg")

 .attr("width", width + margin.left + margin.right)

 .attr("height", height \* 1 + margin.top + margin.bottom)

 .append("g")

 .attr("class", "graph")

 .attr("transform", "translate(" + margin.left + "," + margin.top + ")")
;

The rest of the graph is rendered after we get data. Data can be
retrieved using D3’s [fetch
API](https://github.com/d3/d3/blob/master/API.md#fetches-d3-fetch):

d3.tsv("data.tsv", d3.autoType ).then ( function( data) {} ) ;

This fetch method retrieves a tab separated values file. There are
similar methods for retrieving CSVs, JSON files, other delimiters, and
opaque blobs. The first parameter is the file to retrieve, a relative or
full URL. The second parameter is called on each line (row) that is
received, determining the data type of each column. Here I am using a D3
utility *autoType()* . Finally, a function is called on the entire data.
This is done using Javascript Promise API (in recent versions ; in
earlier versions the callback function was a parameter to the fetch
function). The fetch operation is done asynchronously, So the *d3.tsv()*
call returns immediately, returning a Promise object. The Promise object
implements a *then()* method that is called when the data is ready. The
Promise API provides a clean way to handle errors, especially useful
when a series of fetches is done. See [What does the function then()
mean in JavaScript? - Stack
Overflow](https://stackoverflow.com/questions/3884281/what-does-the-function-then-mean-in-javascript)
for more details.

The data passed to the callback function is an Array of objects, one
object for each line. Each object has an attribute for each column in
the file. The body of the callback will be filled out below.

First, set the domain of each scale:

 y.domain(\[0, d3.max(data, function(d) {

 return d.frequency;

 })\]);

 x.domain(data.map(function(d) {

 return d.letter;

 }));

Next, use the canvas defined above (the translated portion) to add the x
and y axis. Note that the X axis has been positioned at the bottom of
the screen. Without that translation, it will be at the top of the
screen (drawn at the origin). Some styling has been done on both lines.

 var svg = d3.selectAll(".graph") ;

 svg.append("g")

 .attr("transform", "translate(0," + height + ")")

 .call(xAxis)

 .style('fill', 'none')

 .style('stroke', '\#000')

 ;

 svg.append("g")

 .call(yAxis)

 .style('fill', 'none')

 .style('stroke', '\#000')

 ;

Now we are ready for binding to the data. As this is a static graph,
only the *enter()* section is present:

svg.selectAll(".graphobj")

 .data(data)

 .enter()

 .append("circle")

 .attr("class", "graphobj")

 .attr("cx", function(d) {

 return x(d.letter);

 })

 .attr("cy", function(d) {

 return y(d.frequency);

 })

 .attr("r", 5 )

 .attr("fill", function(d, i) {

 return color(i);

 })

 .attr("id", function(d, i) {

 return i;

 })

 ;

And there you have it, the data is graphed. For each data element, a
[circle](https://www.w3schools.com/graphics/svg_circle.asp) is drawn. A
circle is an SVG element defined in the HTML/SVG specification. The
scale functions defined earlier (*x* and *y*) are used to get the x and
y coordinates of each data item. The color of each item is set using a
color scale, and here is a function of its index.

One final note, I used the full function syntax here for clarity. There
is an abbreviated syntax that could be used:

svg.selectAll(".circle")

 .data(data)

 .enter()

 .append("circle")

 .attr("class", "circle")

 .attr("cx", d =&gt; { return x(d.letter) })

 .attr("cy", d =&gt; { return y(d.frequency) } )

 .attr("r", 5 )

 .attr("fill", (d,i ) =&gt; { return color(i) })

 .attr("id", (d,i ) =&gt; { return i })

 ;

Other primitives
----------------

The above example drew a basic SVG element, a circle, on the canvas.
What about more complicated shapes? While you could draw a line segment
for each data element, there is a better way. You use an SVG element
called a [path](https://www.w3schools.com/graphics/svg_path.asp). D3 has
utility functions for rendering
[shapes](https://github.com/d3/d3/blob/master/API.md#shapes-d3-shape)
using paths. We will be using the
[line](https://github.com/d3/d3/blob/master/API.md#lines) shape. First,
thse utilitiy functions have to be added to the initialization:

 &lt;script src="https://d3js.org/d3.v5.min.js"&gt;&lt;/script&gt;

 &lt;script src="https://d3js.org/d3-path.v1.min.js"&gt;&lt;/script&gt;

 &lt;script src="https://d3js.org/d3-shape.v1.min.js"&gt;&lt;/script&gt;

The path element uses a series of directions to render the path. To
create these directions, a function has to be defined that generates
them from your data

var line = d3.line()

 .x(function(d) { return x(d.letter); })

 .y(function(d) { return y(d.frequency); });

Like scales and axis, this statement is returning a function. This
definition can be located anywhere in the script. To take these
directions and render the path, a path element has to be added to the
canvas. This is done inside the data callback:

svg.append("path")

 .attr("d", line(data))

 .attr("stroke", "black")

 .attr("fill", "none") ;

The path directions are specified using the d attribute, set by the
output of the function that has been defined. The fill and stroke
attributes are set as the defaults have the path filled in.

The *line()* function computes a path over the whole data. Run it
manually on the data, and you should see something like this:

"M5,89.25759722878284L25,220.63454574082823L45,195.2448433317588L65,166.29270980947882L85,0L105,204.96772161864274L125,210.34089119823648L145,130.0582585419619L165,112.89560699102502L185,246.98866320264526L205,234.80554243426232L225,170.78019209573296L245,202.6452527161077L265,117.16658793890727L285,102.24767753109745L305,212.0335380255078L325,248.13021571406077L345,132.16422610612503L365,125.47236655644778L385,71.76035270036213L405,195.71720988820655L425,230.75106282475198L445,203.5506219492993L465,247.04770902220122L485,211.14785073216817L505,248.54353645095262"

Bar Graphs
==========

Scatterplots and line graphs are not the only type of visualization that
can be done in D3. This section covers bar graphs. Some small changes to
the scatterplot visualization will turn it into a bar graph.

Start with a similiar skeleton HTML document:

&lt;!DOCTYPE html&gt;

&lt;html&gt;

&lt;head&gt;

 &lt;meta charset="utf-8"&gt;

 &lt;title&gt;Bar Chart&lt;/title&gt;

 &lt;script src="https://d3js.org/d3.v5.min.js"&gt;&lt;/script&gt;

&lt;/head&gt;

&lt;body&gt;

&lt;/body&gt;

&lt;/html&gt;

As before, this first step is to define the scales. The linear scale
stays the same, but the ordinal scale is changed from *scalePoint()* to
*scaleBand()*. For this example, I am switching the x and y axes. The
letters will be plotted on the y axis, and the frequencies on the x.

var y = d3.scaleBand()

 .rangeRound(\[0, height\] );

var x = d3.scaleLinear()

 .range(\[0, width\]);

Both *scalePoint()* and *scaleBand()* are ordinal scale functions that
place categorical data evenly across the axis. The difference is that
Point returns the center of the interval, while Band returns the leading
edge of the interval. Band also has a method for returning the width of
the interval, which will be used when the bar is drawn.

The margin and axis code is unchanged. Also unchanged is the code for
data fetching, setting the domain for the scales, and the rendering of
the axis. What is different is what is rendered when the data is bound.
Instead of drawing circles, a bar for each data item will be drawn. The
SVG element used is a
[rectangle](https://www.w3schools.com/graphics/svg_rect.asp) (**rect**).
Here is the code:

 var elements = svg.selectAll(".graphobj")

 .data(data)

 .enter()

 .append("rect")

 .attr("class", "graphobj")

 .attr("y", function(d) {

 return y(d.letter);

 })

 .attr("height", y.bandwidth())

 .attr("x", 0)

 .attr("width", function(d) {

 return x(d.frequency);

 })

 .attr("fill", function(d, i) {

 return color(i);

 })

 .attr("id", function(d, i) {

 return i;

 })

 ;

As with the scatterplot, the x and y locations are set. The x location
is 0, because the rectangle starts at the left edge of the plot. Recall
that the (0,0) location is the top left of the screen. Additionally, the
height and width have to be set. The height is the size of the interval
allocated for that data item, given by the scale method *bandwidth()*.
The width is the length of the bar, which is the frequency scaled to
pixel values, returned by calling the linear scale function previously
defined. The fill and id attribute code is unchanged.

A good excerise to do at this point would be to switch the x and y axis
on this bar graph. This will give you a good feel for how scales work
and how they interact with your data.

Grouping data
-------------

Often it is not the raw data that is visualized in a bar chart, but some
aggregation of it. D3 has a grouping function, somewhat analogous to
PANDAS groupby. In D3, the groupby function is called *nest*. To specify
what to groupby, a function is passed to the nest object’s *keys()*
method. Here is an example, using [network flow
data](https://www.auvik.com/franklymsp/blog/netflow-basics/) (the data
has been filtered so that only client to server records are present):

d3.nest()

 .key(function(d) {return d.proto + ":" + d.dport} ) ;

Note that the key is not one of the original columns, but a new value
derived from them. To apply the nest object to the data, use the
*entries()* method. E.g.

d3.nest()

 .key(function(d) {return d.proto + ":" + d.dport} )

 .entries(data);

At this point, no aggregation is done. The output is then an array
containing the original objects grouped by the key value. E.g.

\(6) \[…\]

To do an aggregation, the *rollup()* method is used. It specifies an
aggregation function. Here is an example where a count aggregation is
done:

var counts = d3.nest()

 .key(function(d) {return d.proto + ":" + d.sport} )

 .rollup(function(v) { return v.length; })

 .entries(data);

The result is another array, but with the aggregated values:

\(6) \[…\]

With this aggregation, a bar chart can be drawn. First, each scale’s
domain has to be set:

 x.domain(\[0, d3.max(counts, function(d) {

 return d.value;

 })\]);

 y.domain(counts.map(function(d) {

 return d.key;

 }));

Here we are using Javascript’s *array.*map() function to get an array of
keys. Now the bars can be rendered:

 var elements = svg.selectAll(".graphobj")

 .data(counts)

 .enter()

 .append("rect")

 .attr("class", "graphobj")

 .attr("y", function(d) {

 return y(d.key);

 })

 .attr("height", y.rangeBand())

 .attr("width", function(d) {

 return x(d.value);

 })

 .attr("x", function(d) {

 return 0;

 })

 .attr("fill", function(d, i) {

 return color(i);

 })

 .attr("id", function(d, i) {

 return i;

 })

Because of the skewness of the data, it is best visualized at log scale.
First, change the scale:

var x = d3.scaleLog()

 .range(\[0, width\]);

You also need to change the domain, as log plots the domain start at 1,
not 0:

 x.domain(\[1, d3.max(counts, function(d) {

 return d.value;

 })\]);

While not strictly necessary, the graph will be easier to read if you
explicitly define the tick label formatting. By default, exponentials
are displayed which I find hard to interpret:

var xAxis = d3.axisBottom()

 .scale(x)

 .ticks(3,"g") ;

Stacked Bar Charts
------------------

Multiple levels of nesting can be defined by specifying multiple keys.
E.g.

var nested = d3.nest()

 .key(function(d) {return d.proto + ":" + d.sport} )

 .key(function(d) {return d.sip} )

 .rollup(function(v) { return v.length; })

 .entries(data);

This kind of structure can be visualized using a stacked bar chart. D3
does have a helper function,
[d3.](https://github.com/d3/d3/blob/master/API.md#stacks)[*stack()*](https://github.com/d3/d3/blob/master/API.md#stacks)
which can be used to stack data. However, it expects a dense (completely
filled out) two dimensional array as its input, and not a nested
structure. But it is not hard to write a function that takes *nest()*’s
output and turns it into a stack:

function nestToStack(nest) {

 var stacked = \[\] ;

 nest.forEach( function(d) {

 var x0 = 0 ;

 d.values.forEach( function(f) {

 var obj = { y: d.key } ;

 obj.x = \[ x0, x0+ f.value \] ;

 x0 += f.value ;

 obj.label = f.key ;

 stacked.push(obj) ;

 } ) ;

 } );

 return stacked ;

}

With this utility, we can stack our nested data and set the domains:

 var counts = nestToStack(nested) ;

 x.domain(\[0, d3.max(counts, function(d) {

 return d.x\[1\];

 })\]);

 y.domain(counts.map(function(d) {

 return d.y;

 }));

Now we are ready to render the graph:

 var elements = svg.selectAll(".graphobj")

 .data(counts)

 .enter()

 .append("rect")

 .attr("class", "graphobj")

 .attr("y", function(d) {

 return y(d.y);

 })

 .attr("height", y.bandwidth())

 .attr("width", function(d) {

 return x(d.x\[1\])-x(d.x\[0\]);

 })

 .attr("x", function(d) {

 return x(d.x\[0\]);

 })

 .attr("fill", function(d, i) {

 return color(d.label);

 })

 .attr("id", function(d, i) {

 return i;

 }) ;

This chart is still skewed, but it is not appropriate to display a
stacked chart at log scale.

Callbacks
=========

A hallmark of D3 is interactive visualizations. This is accomplished via
callbacks. A callback can be registered to any selection using the
[*selection.*](https://github.com/d3/d3-selection/blob/v1.4.1/README.md#selection_on)[on()](https://github.com/d3/d3-selection/blob/v1.4.1/README.md#selection_on)
method. Any [DOM event
type](https://developer.mozilla.org/en-US/docs/Web/Events#Standard_events)
can be listened to. Here are some common ones:

-   click : mouse click
-   mouseover
-   mouseout
-   mousemove

Here is an example callback that changes the color of the element the
mouse is over. The *if* statement in the **mouseout** is there so that
the code will work with the nested graph. In all the other examples, the
color is set by the id.

d3.selectAll(“.graphobj”)

 .on("mouseover", function(d) {

 d3.select(this)

 .attr("fill", "red");

 })

 .on("mouseout", function(d, i) {

 d3.select(this).attr("fill", function() {

 var fill ;

 if (‘ylabel’ in d) {

 fill = color(d.ylabel)

 }

 else {

 fill = color(this.id)

 }

 return fill ;

 });

It is possible to attach a callback to the canvas. You do have to make
sure that it is attached to the SVG object, and not the translated
version of it:

E.g.:

function my\_callback() { alert(“Clicked on canvas”) }

var svg = d3.select("body").append("svg")

 .attr("width", width + margin.left + margin.right)

 .attr("height", height \* 1 + margin.top + margin.bottom)

 .on("click", my\_callback)

 .append("g")

 .attr("class", "graph")

 .attr("transform", "translate(" + margin.left + "," + margin.top + ")")
;

D3 has a method for getting the mouse coordinates on the canvas, which
will be in pixels. Before you can map it back to your data, you do have
to account for the shifting:

function my\_callback () {

 var coordinates= d3.mouse(this);

 var yval = y.invert(coordinates\[1\]-margin.top) ;

 alert(

 "y " + yval +

 "\\ncanvas: " + coordinates\[0\] + "," + coordinates\[1\] +

) ;

 }

You can also get the mouse position on the page. Here is how that is
done:

function my\_callback () {

 var coordinates= d3.mouse(this);

 var yval = y.invert(coordinates\[1\]-margin.top) ;

 alert(

 "y " + yval +

 "\\ncanvas: " + coordinates\[0\] + "," + coordinates\[1\] +

 "\\npage = " + d3.event.pageX +"," + d3.event.pageY

) ;

 }

The page location can be used to create tooltips using HTML page
elements. See [Simple d3.js tooltips –
bl.ocks.org](https://bl.ocks.org/d3noob/a22c42db65eb00d4e369) for an
example.

Additional utilities
====================

This tutorial covered a small fraction of the [D3
API](https://github.com/d3/d3/blob/master/API.md). It also has:

-   [Numeric
    histograms](https://github.com/d3/d3-array/blob/v1.2.4/README.md#histograms)
-   [Time
    parsing](https://github.com/d3/d3/blob/master/API.md#time-formats-d3-time-format)
-   [Force directed
    graphs](https://github.com/d3/d3/blob/master/API.md#forces-d3-force)
-   [Pan and
    zooming](https://github.com/d3/d3/blob/master/API.md#zooming-d3-zoom)
-   [Animations](https://github.com/d3/d3/blob/master/API.md#transitions-d3-transition)

And much more.
