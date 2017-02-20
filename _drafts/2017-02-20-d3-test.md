---
title: Box-Muller transform visualization
layout: post
comments: true
date: 2017-02-19
---
<style>

.bar rect {
  fill: steelblue;
}

.bar text {
  fill: #fff;
  font: 10px sans-serif;
}

</style>
<div id="chart"></div>
<script src="https://d3js.org/d3.v4.min.js"></script>
<script>

function random_normal(mean, variance) {
	var z0, r, theta;
	r = Math.sqrt(-2 * Math.log(Math.random()))
	theta = 2 * Math.PI * Math.random()
	z0 = r * Math.cos(theta)

	return mean + variance * z0
}

var data = [];
for (var i = 0; i < 50000; i++) {
    data.push(random_normal(0, 1));
}

var WIDTH = 1000;
var HEIGHT = 500;

var xScale = d3.scaleLinear().domain([-3, 3]).range([0, WIDTH]);
var binnedData = d3.histogram().domain(xScale.domain()).thresholds(xScale.ticks(20))(data);
var yScale = d3.scaleLinear().domain([0, d3.max(binnedData, function(d) { return d.length; })]).range([HEIGHT, 0]);

var svg = d3.select("#chart").append("svg")
                .attr("currentScale", 1)
                .attr("viewBox", "0 0 1000 500");

var g = svg.append("g");

var bar = g.selectAll("g.bar").data(binnedData).enter()
                .append("g")
                .classed("bar", true)
                .attr('transform', function(d) { return 'translate(' + xScale(d.x0) + ',' + 0 + ')'});

bar.append("rect")
    .attr("x", 1)
    .attr("y", HEIGHT)
    .attr("width", xScale(binnedData[0].x1) - xScale(binnedData[0].x0) - 0.5)
    .transition().duration(500)
    .attr("y", function(d) { return yScale(d.length) })
    .attr("height", function(d) { return HEIGHT - yScale(d.length); });

bar.append("text")
    .attr("dy", ".75em")
    .attr("y", HEIGHT)
    .attr("dy", 15)
    .attr("x", (xScale(binnedData[0].x1) - xScale(binnedData[0].x0)) / 2)
    .attr("text-anchor", "middle")
    .text(function(d) { return d.length; })
    .transition().duration(500)
    .attr("y", function(d) { return yScale(d.length) });

</script>

Box-Muller transform을 이용하여 총 50000개의 난수를 발생시키고 histogram을 그려보았다.

## Code

{% highlight html %}
<style>

.bar rect {
  fill: steelblue;
}

.bar text {
  fill: #fff;
  font: 10px sans-serif;
}

</style>
<div id="chart"></div>
<script src="https://d3js.org/d3.v4.min.js"></script>
<script>

function random_normal(mean, variance) {
    var z0, r, theta;
    r = Math.sqrt(-2 * Math.log(Math.random()))
    theta = 2 * Math.PI * Math.random()
    z0 = r * Math.cos(theta)

    return mean + variance * z0
}

var data = [];
for (var i = 0; i < 50000; i++) {
    data.push(random_normal(0, 1));
}

var WIDTH = 1000;
var HEIGHT = 500;

var xScale = d3.scaleLinear().domain([-3, 3]).range([0, WIDTH]);
var binnedData = d3.histogram().domain(xScale.domain()).thresholds(xScale.ticks(20))(data);
var yScale = d3.scaleLinear().domain([0, d3.max(binnedData, function(d) { return d.length; })]).range([HEIGHT, 0]);

var svg = d3.select("#chart").append("svg")
                .attr("currentScale", 1)
                .attr("viewBox", "0 0 1000 500");

var g = svg.append("g");

var bar = g.selectAll("g.bar").data(binnedData).enter()
                .append("g")
                .classed("bar", true)
                .attr('transform', function(d) { return 'translate(' + xScale(d.x0) + ',' + 0 + ')'});

bar.append("rect")
    .attr("x", 1)
    .attr("y", HEIGHT)
    .attr("width", xScale(binnedData[0].x1) - xScale(binnedData[0].x0) - 0.5)
    .transition().duration(500)
    .attr("y", function(d) { return yScale(d.length) })
    .attr("height", function(d) { return HEIGHT - yScale(d.length); });

bar.append("text")
    .attr("dy", ".75em")
    .attr("y", HEIGHT)
    .attr("dy", 15)
    .attr("x", (xScale(binnedData[0].x1) - xScale(binnedData[0].x0)) / 2)
    .attr("text-anchor", "middle")
    .text(function(d) { return d.length; })
    .transition().duration(500)
    .attr("y", function(d) { return yScale(d.length) });

</script>
{% endhighlight %}