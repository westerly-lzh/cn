---
layout: post
title: D3 Tutorials Basic Shapes : Circle
categories:
- D3 Tutorials
tags:
- D3
- Basic Shapes

---

<script src="http://d3js.org/d3.v3.min.js" charset="utf-8"></script>
<style>
div{
	float:left;
}
</style>
<div>
<input type="button" value="Dismiss" name="Dimiss" onclick="dismiss()"></input>
<input type="button" value="Show" name="Show" onclick="show()"></input>
<svg id="circles1"></svg>
<script type="text/javascript">
var data=[{x:20,y:40,r:5},{x:30,y:60,r:10},{x:40,y:80,r:15},{x:50,y:100,r:20}];
var circle = d3.select("#circles1").selectAll("circle").data(data);
circle.enter().append("circle")
	.attr("cx",function(d){return d.x;})
	.attr("cy",function(d){return d.y;})
	.attr("r",function(d){return d.r;});

function dismiss(){
	d3.selectAll("circle").transition().attr("r",0).remove();
}

function show(){
	var circle = d3.select("#circles1").selectAll("circle").data(data);
	circle.enter().append("circle")
	.attr("cx",function(d){return d.x;})
	.attr("cy",function(d){return d.y;})
	.attr("r",0);
	d3.selectAll("circle").data(data).transition().attr("r",function(d){return d.r;});
}
</script>
</div>
<div>
<svg id="circles2"></svg>
<script type="text/javascript">

</script>
</div>
