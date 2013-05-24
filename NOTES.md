D3 PRESENTATION
===============


## SVG

Scalable Vector Graphics (SVG) is an XML-based vector image format for two-dimensional graphics that has support for interactivity and animation. The SVG specification is an open standard developed by the World Wide Web Consortium (W3C) since 1999.

[Wikipedia][http://en.wikipedia.org/wiki/Scalable_Vector_Graphics]

What we need to know is:
* XML based, tags
* optimized for images => better for visualizations


## SVG in the DOM

    <svg width="700" height="500">
      <rect x="0" width="5" y="0" height="50"></rect>
      <rect x="0" width="31" y="70" height="50"></rect>
      <rect x="0" width="86" y="140" height="50"></rect>
      <rect x="0" width="474" y="210" height="50"></rect>
      <rect x="0" width="308" y="280" height="50"></rect>
      <rect x="0" width="700" y="350" height="50"></rect>
      <rect x="0" width="630" y="420" height="50"></rect>
    </svg>


## Insert SVG into DOM


    var canvas_d = {width: 700, height: 500},
  
    canvas = d3.select('#canvas').append('svg')
        .attr('width', canvas_d.width)
        .attr('height', canvas_d.height);


## Insert content into SVG based on data

First attempt would be to iterate over each item in the data and append
a `rect`.

Let's see in the source what varies:
* width
* y

With some math we can do

    width = canvas_d.width * (d/max_overall)

    y = 70  * i

We use d3 to get overall max

    var data = [26009896, 179804755, 494478797, 2718505888, 1765686465, 4015692380, 3611612096],

    max_overall = d3.max(data);

Ok, so all together...

    data.map(function(d, i) {

      canvas.append('rect')
       .attr('x', 0 )
       .attr('width', canvas_d.width * (d/max_overall))
       .attr('y', 70 * i)
       .attr('height', 50)

    });


Some of the expressions can also be written as self executing anonymous
functions.

    data.map(function(d, i) {
      canvas.append('rect')
       .attr('x', 0 )
       .attr('width', (function(d, i) {
         return canvas_d.width * (d/max_overall)
       })(d, i))
       .attr('y', (function(d, i) {
         return 70 * i;
       })(d, i))
       .attr('height', 50)
    });

## Insert content the d3 way

Let's look into general before we dive into details.

    canvas.selectAll('rect')
        .data( data )
      .enter().append('rect')
        .attr('x', 0 )
        .attr('width', function(d, i){
          return canvas_d.width * (d/max_overall);
        })
        .attr('y',  function(d, i) {
          return 70 * i;
        })
        .attr('height', 50)

We make a selection of rect elements in our svg

    canvas.selectAll('rect')

Than we bind data to it using a d3 "data join"

    .data( data )

We call enter(), just to get the collection of all data items, which
have no corresponding element. For this sub collection, we actually
append `rect` elements (create them).

    .enter().append('rect')

On this collection we do

    .attr('x', 0 )
    .attr('width', function(d, i){
      return canvas_d.width * (d/max_overall);
    })
    .attr('y',  function(d, i) {
      return 70 * i;
    })
    .attr('height', 50)

Which is pretty much the same, except the functions are no longer self
executing. D3 will check if the value is collable and will do so, always
passing the data and index.

This is often just written as

    canvas.selectAll('rect')
        .data( data )
      .enter().append('rect')
        .attr('x', 0)
        .attr('width', function(d, i){ return canvas_d.width * (d/max_overall) })
        .attr('y',  function(d, i) { return 70 * i })
        .attr('height', 50);


## What are does data joins?

Collections of SVG elements can be joined with data to create new collections.
By calling some methods we can get subollections such as:
* all new elements (data items with no correspoding element)
* items to be removed (elements with no corresponding data element)

To get an idea take the following html

    <svg width="700" height="500">
      <circle cx="100" cy="250" r="30"></circle>
      <circle cx="300" cy="250" r="30"></circle>
      <circle cx="500" cy="250" r="30"></circle>

We want to join these elements with some data:

    var data = [100, 200, 300, 400, 500, 600]

    var canvas = d3.select('svg');

    // Collection of elements joined with data
    var circles = canvas.selectAll('circle')
      .data( data );

    // Act on current collection of elements
    circles
        .attr('r', 40);

    // Act on collection of new elements
    circles.enter().append('circle')
        .attr('cx', function(d, i){ return d })
        .attr('cy', 250)
        .attr('r', 0)
        .style('fill', 'green')
        .attr('r', 15);

    // Act on current collection of items
    circles
        .style('fill', 'blue');

    // If we changed data to fewer items,
    // we will also have a collection of items to be removed

    // Act on items to be removed
    circles.exit()
        .attr('r', 5)
        .remove();

What we need to remember:
* We select items from within our SVG element
* We create a data joing with data( data )
* We can act on the existing or updated items
* We can act on new items, these should be created and appended
* We can act on old items, these should be removed

Back to our visualization:

    canvas.selectAll('rect')
        .data( data )
      .enter().append('rect')
        .attr('x', 0)
        .attr('width', function(d, i){ return canvas_d.width * (d/max_overall) })
        .attr('y',  function(d, i) { return 70 * i })
        .attr('height', 50);

What we have so far:
* Insert SVG container (canvas)
* Create new elements via data binding
* Set attributes to those new items


## Add margin

If we want to put labels, we'll need some whitespace.

First, create a chart_d which are in function of the margin.

    margin = {top: 50, right: 20, bottom: 20, left: 100},
    canvas_d = {width: 700, height: 500},
    chart_d = {
      width: canvas_d.width -  margin.right - margin.left,
      height: canvas_d.height - margin.top - margin.bottom
    },

Actually create a "chart"

    chart = canvas.append('g')
        .attr( 'transform', 'translate('+ margin.left +', '+ margin.top +')' )
          .attr( 'width' , chart_d.width)
          .attr('height', chart_d.height);

Finally update rect code to use chart and chart dimensions instead.

    chart.selectAll('rect')
        .data( data )
      .enter().append('rect')
        .attr('x', 0)
        .attr('width', function(d, i){ return chart_d.width * (d/max_overall) })
        .attr('y',  function(d, i) { return 60 * i })
        .attr('height', 50)


## Linear scale

The ratio math in our code looks ugly.

D3 helps us with that in the form of "scales".

There are three main categories of scales:
* time scale - for time domains.
* ordinal scales -  for discrete input domains, such as names or categories.
* quantitative scales - for continuous input domains, such as numbers.

A scale convert some input into some output.

    width:  data => pixels

    [26009896, 179804755, 494478797, 2718505888, 1765686465, 4015692380, 3611612096]

    <rect x="0" width="5" y="0" height="50"></rect>
    <rect x="0" width="31" y="70" height="50"></rect>
    <rect x="0" width="86" y="140" height="50"></rect>
    <rect x="0" width="474" y="210" height="50"></rect>
    <rect x="0" width="308" y="280" height="50"></rect>
    <rect x="0" width="700" y="350" height="50"></rect>
    <rect x="0" width="630" y="420" height="50"></rect>


    26009896           =>              5
    179804755          =>             31
    494478797          =>             86
    2718505888         =>            474
    1765686465         =>            308
    4015692380         =>            700
    3611612096         =>            630


    var width = scaleFunc(179804755)     // 31

First, create scale

    x = d3.scale.linear()
      .domain([0, max_overall])
      .range([0, chart_d.width]);

Use it to convert width value

    chart.selectAll('rect')
        .data( data )
      .enter().append('rect')
        .attr('x', 0)
        .attr('width', function(d, i){ return x(d) })
        .attr('y',  function(d, i) { return 60 * i })
        .attr('height', 50);

The shorthand would be this.

    chart.selectAll('rect')
        .data( data )
      .enter().append('rect')
        .attr('x', 0)
        .attr('width', x)
        .attr('y',  function(d, i) { return 60 * i })
        .attr('height', 50);

## Ordinal scales

We do some other math too, though it's not as challenging.

    y:  data-item => pixels

    [26009896, 179804755, 494478797, 2718505888, 1765686465, 4015692380, 3611612096]

    <rect x="0" width="5" y="0" height="50"></rect>
    <rect x="0" width="31" y="70" height="50"></rect>
    <rect x="0" width="86" y="140" height="50"></rect>
    <rect x="0" width="474" y="210" height="50"></rect>
    <rect x="0" width="308" y="280" height="50"></rect>
    <rect x="0" width="700" y="350" height="50"></rect>
    <rect x="0" width="630" y="420" height="50"></rect>


    26009896           =>              0
    179804755          =>             70
    494478797          =>            140
    2718505888         =>            210
    1765686465         =>            280
    4015692380         =>            350
    3611612096         =>            420

    var y = someOtherScaleFunc(179804755)     // 70

First, create scale

    y = d3.scale.ordinal()
        .domain(data)
        .rangeBands([0, chart_d.height]);

Notice:
* We pass all the values, this is because we see them as individual items
* We use rangeBands instead of range

Use it to calculate y

    chart.selectAll('rect')
        .data( data )
      .enter().append('rect')
        .attr('x', 0)
        .attr('width', x)
        .attr('y', y)
        .attr('height', 50);

But event better, we can use that scale to get the height

    chart.selectAll('rect')
        .data( data )
      .enter().append('rect')
        .attr('x', 0)
        .attr('width', x)
        .attr('y', y)
        .attr('height', y.rangeBand);

## Labels and axis

All this work, and it still looks the same.

Labels are automatically created by the axis. So let's create one.

Creating axis is a two step process:
* axis creator function
* draw axis

Axis Creator function

    axis_y_f = d3.svg.axis()
        .scale(y)
        .orient('left')

Draw it

    axis_y = chart.append('g')
        .attr('class', 'axis y')
        .call(axis_y_f);

We can make the axis prettier

    axis_y_f = d3.svg.axis()
        .scale(y)
        .orient('left')
        .tickPadding(30)
        .tickSize(0, 0, 0);



Got it:
* We need scales if we want axis
* Axis creation is a two step process
* If we want to tune the axis, we probably need to be in the creator
  function

## And for the X axis

The same story...

    axis_x_f = d3.svg.axis()
        .scale(x)
        .orient('top')

    axis_x = chart.append('g')
        .attr('class', 'axis x')
        .attr( 'transform', 'translate('+ 0 +', '+ 0 +')' )
        .call(axis_x_f);

Add nicer ticks

    axis_x_f = d3.svg.axis()
        .scale(x)
        .orient('top')
        .tickPadding(10)
        .ticks(5)
        .tickSize(chart_d.height, chart_d.height, 0)

    axis_x = chart.append('g')
        .attr('class', 'axis x')
        .attr( 'transform', 'translate('+ 0 +', '+ chart_d.height +')' )
        .call(axis_x_f);

Add subticks

    axis_x_f = d3.svg.axis()
        .scale(x)
        .orient('top')
        .tickPadding(10)
        .ticks(5)
        .tickSize(chart_d.height, chart_d.height, 0)
        .tickSubdivide(2)


## Add real labels

Let's change the data source so we can have real labels.

    var data = [{value: 26009896, name: "angular.js"},
                {value: 179804755,  name: "backbone.js"},
                {value: 494478797,  name: "batman.js"},
                {value: 2718505888,  name: "ember.js"},
                {value: 1765686465,  name: "knockout.js"},
                {value: 4015692380,  name: "sammy.js"},
                {value: 3611612096, name: "spine.js"}];

For everything to work, we need to make some modifications, more
specifically, we need data accessor functions.

Just so nothing really breaks...

    max_overall = d3.max(data, function(d, i) { return d.value }),

    chart.selectAll('rect')
        .data( data )
      .enter().append('rect')
        .attr('x', 0)
        .attr('width', function(d, i) { return x(d.value) })
        .attr('y', function(d, i) { return y(d.name) })
        .attr('height', y.rangeBand);

And now for the labels

    y = d3.scale.ordinal()
        .domain(data.map(function(d, i){ return d.name }))
        .rangeBands([0, chart_d.height], 0.1);

We might need to change the padding.


## Animate

Everything is so static...

One of the nice things of d3.js are the animations. So let's jump into it.

First, animate the bars' width

    chart.selectAll('rect')
        .data( data )
      .enter().append('rect')
        .attr('x', 0)
        .attr('width', 0)
        .attr('y', function(d, i) { return y(d.name) })
        .attr('height', y.rangeBand)
        .style('fill', '#333')
      .transition()
        .duration(600)
        .attr('width', function(d, i) { return x(d.value) })

Chain a color transition.

    chart.selectAll('rect')
        .data( data )
      .enter().append('rect')
        .attr('x', 0)
        .attr('width', 0)
        .attr('y', function(d, i) { return y(d.name) })
        .attr('height', y.rangeBand)
        .style('fill', '#333')
      .transition()
        .duration(600)
        .attr('width', function(d, i) { return x(d.value) })
      .transition()
        .duration(400)
        .style('fill', 'black')

Animation can be done on about anything. Axis too...

    axis_y = chart.append('g')
        .attr('class', 'axis y')
      .transition()
        .delay(800)
        .duration(800)
        .call(axis_y_f);

## Timely updates

Our data changes every x seconds... we better update the chart.

We've changed the data source so that:
* drawchart gets called once, the first time
* every x seconds, updateChart gets called with new data


Asign the bars to a var so we can access it from another function.

    bars = chart.selectAll('rect')
        .data( data )
      .enter().append('rect')
        .attr('x', 0)
        .attr('width', 0)
        .attr('y', function(d, i) { return y(d.name) })
        .attr('height', y.rangeBand)u
        .style('fill', '#333');
  
    bars
      .transition()
        .duration(600)
        .attr('width', function(d, i) { return x(d.value) })
      .transition()
        .duration(400)
        .style('fill', 'black')

Our update function.

    function updateChart(data) {
    
      bars
          .data(data)
        .transition()
          .duration(600)
          .attr('width', function(d, i) { return x(d.value) });
    
    }

Animations make the updates easier to follow.

## Updating the entire axis

What's with all that unused whitespace?

It's not only the chart content that can be updated.

To update the axis, just update the scale and call something on the axis
again.

    function updateChart(data) {
  
      // Set the current overal max;
      max_overall = d3.max(data, function(d, i) { return d.value });
  
      // Update the bars with new data
      bars
          .data(data)
        .transition()
          .delay(800)
          .duration(600)
          .attr('width', function(d, i) { return x(d.value) });
  
      // Update the x scale
      x.domain([0, max_overall])
  
      // Update x axis
      axis_x
        .transition()
          .duration(600)
          .call(axis_x_f);
  
      // Update the bars
      bars
        .transition()
          .duration(600)
          .attr('width', function(d, i) { return x(d.value) });
    }


Done

Recap