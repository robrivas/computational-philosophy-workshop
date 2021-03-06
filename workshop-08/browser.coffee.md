# Browser Code

This is the browser file where we pull together different parts of the simulation and wrap it up in the browser.  We'll start out by importing our libraries and defining any global variables we might need.


    d3 = require '../assets/d3.min.js'
    simulation = require './simulation.coffee.md'
    running = false
    height = window.innerHeight || 600
    width = window.innerWidth || 600
    space = null



Next, we'll grab create our svg canvas, apply some event listeners and add it to the DOM.


    canvas = d3.select("#space")
          .append("svg:svg")
          .attr("height", height)
          .attr("width", width)
          .on "click", () ->
            running = if running is false then true else false



Let's add a splash of colour.  This function will take an agent data point and return a hex colour code on the blue-red spectrum based on their belief.  The HTML hex code is #RRGGBB where each red-green-blue value is a base16 transformation of 0-255.


    colourise = (d) ->
      red = Math.floor((1-d.credence)*255).toString 16
      blue = Math.floor(d.credence*255).toString 16
      red = "0#{red}" if red.length is 1
      blue = "0#{blue}" if blue.length is 1 
      "##{red}00#{blue}"


Now we need to createsvg circles to represent our agents and bind them to the actual agents from the simulation.  The `d` in the functions here is the D3js accessor to the agent data.

  
    populate = () ->
      space = simulation.create height, width
      canvas.selectAll "circle"
        .data space
        .enter().append "circle"
        .style "fill", (d) -> colourise(d)
        .style "opacity", 0.5
        .attr "r", 8
        .attr "cx", (d) -> d.x
        .attr "cy", (d) -> d.y
        .attr "title", (d) -> "#{d.credence}"

    populate()


We then write a loop where each svg circle triggers the move function for its bound agent.  


    move = () ->
      circles = canvas.selectAll "circle"
      circles.each (d) ->
        d = simulation.interact d
      .transition()
      .duration 500
      .attr "cx", (d) -> d.x
      .attr "cy", (d) -> d.y
      .attr "title", (d) -> "#{d.credence}"
      .style "fill", (d) -> colourise d
    

    run = () ->
      move() unless running is false


Finally, we run the loop continuous with a half second pause.


    setInterval run, 1000