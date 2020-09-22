---
title: Analyzing Suicide rates 1985-2016 with D3.js
author: Learning Monk
date: '2020-09-18'
categories:
  - Data Visualization
tags:
  - D3.js
  - data-viz
slug: analyzing-suicide-rates-1985-2016
draft: no
---

In this article, I am going to create a simple **Stacked bar chart** using ***D3.js (a Javascript library for data visualization)*** to understand suicide rates by Region and Gender.

Here is the link to the [dataset](https://www.kaggle.com/russellyates88/suicide-rates-overview-1985-to-2016).

You might ask me, why use D3.js for creating such simple charts, as much easier tools are available. You are right. D3.js is an overkill for such a simple chart. But, the reason behind using D3.js is, I like to create charts from scratch with code, and D3.js helps me learn intricacies of building charts unlike the commercial tools which gives me ready-made templates to work on.

### My goal here is to understand how each piece is put together to create charts in D3.js.

Now, let's get started with building our stacked bar chart. By the way, you can choose any other chart which suits the question at hand. I have chosen stacked bar chart as I can analyze suicides by region and gender in one chart. Also, bar charts are good for analyzing **categorical data**. *Feel free to experiment with other relevant chart types.* 

*On a side note, I will be using Visual Studio code (a code editor).* Feel free to use code editor of your choice and comfort.

[Github project link](https://github.com/learning-monk/DataVisualizationProjects_JS/tree/master/DataViz_with_D3js/Stacked%20bar%20chart%20-%20suicides%20by%20region)

The first step is to create a blank HTML file and load D3.js library. Let's name this file **index.html**. 

## HTML file

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Stacked bar chart</title>
    <!-- Load D3.js library -->
    <script src="https://d3js.org/d3.v5.min.js"></script>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <div id="tooltip" class="hidden">
      <p>Region: <span id="region"></span></p>
      <p>Gender: <span id="gender"></span></p>
      <p>Suicides: <span id="suicides"></span></p>
    </div>
    <script src="stackedbar.js"></script>
  </body>
</html>

```

Ok, let's understand the details of this file before moving forward.

- We are loading D3.js library
- Load external CSS file (yet to be created)
- Load a JavaScript file with all the code that is required to create this chart
- Created a *div* element for the tooltip (more on this later)

Now, let's create **JavaScript file** which will host the code for creating the data visualization we want to develop. Let's name this file **stackedbar.js**.

## JavaScript file

```JavaScript
// set svg width and height
const width = 900,
  height = 600;
const margin = { top: 100, right: 100, bottom: 100, left: 100 };
const innerWidth = width - margin.left - margin.right;
const innerHeight = height - margin.top - margin.bottom;

//Load data
d3.csv("https://raw.githubusercontent.com/learning-monk/DataVisualizationProjects_JS/master/DataViz_with_D3js/Stacked%20bar%20chart%20-%20suicides%20by%20region/suicide_rates_1985-2016.csv")
  .then((finalData) => {
    finalData.forEach((d) => {
      d.Region = d.Region;
      d.female = parseFloat(d.female);
      d.male = parseFloat(d.male);
    });

    // set up stack method
    const stack = d3.stack().keys(["female", "male"]);
    // Data, stacked
    const series = stack(finalData);

    // set-up scales
    const xScale = d3
      .scaleBand()
      .domain(finalData.map((d) => d.Region))
      .range([0, innerWidth])
      .padding(0.1);

    const xAxis = d3.axisBottom().scale(xScale);

    const yScale = d3
      .scaleLinear()
      .domain([0, d3.max(series, (d) => d3.max(d, (d) => d[1]))])
      .rangeRound([innerHeight, 0]);

    const yAxis = d3.axisLeft().scale(yScale);

    //Easy colors accessible via a 10-step ordinal scale
    const colors = d3
      .scaleOrdinal()
      .domain(series.map((d) => d.key))
      .range(
        d3
          .quantize((t) => d3.interpolateSpectral(t * 0.8 + 0.1), series.length)
          .reverse()
      )
      .unknown("#ccc");

    //Create SVG element
    const svg = d3
      .select("body")
      .append("svg")
      .attr("viewBox", [0, 0, width, height]);

    const mainG = svg
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    // Add a group for each row of data
    const g = mainG
      .selectAll("g")
      .data(series)
      .enter()
      .append("g")
      .style("fill", (d) => colors(d.key))
      .attr("transform", `translate(0,0)`);

    // Add a rect for each data value
    g.selectAll("rect")
      .data((d) => d)
      .enter()
      .append("rect")
      .attr("x", function (d, i) {
        return xScale(d.data.Region);
      })
      .attr("y", function (d) {
        return yScale(d[1]);
      })
      .attr("height", function (d) {
        return yScale(d[0]) - yScale(d[1]);
      })
      .attr("width", xScale.bandwidth())
      .on("mouseover", function (d) {
        //Get this bar's x/y values, then augment for the tooltip
        let xPosition =
          parseFloat(d3.select(this).attr("x")) + xScale.bandwidth() * 2;
        let yPosition = parseFloat(d3.select(this).attr("y")) + innerHeight / 2;

        // Update the tooltip position and value
        d3.select("#tooltip")
          .style("left", xPosition + "px")
          .style("top", yPosition + "px")
          .select("#region")
          .text(d.data.Region);

        d3.select("#tooltip")
          .select("#gender")
          .text(d[0] === 0 ? "Female" : "Male");

        d3.select("#tooltip")
          .select("#suicides")
          .text(d[1] - d[0]);

        //Show the tooltip
        d3.select("#tooltip").classed("hidden", false);
      })
      .on("mouseout", function () {
        //Hide the tooltip
        d3.select("#tooltip").classed("hidden", true);
      });

    // draw legend
    const legend = mainG
      .append("g")
      .attr("class", "legend")
      .attr("transform", "translate(" + (innerWidth - 110) + "," + 20 + ")")
      .selectAll("g")
      .data(series)
      .enter()
      .append("g");

    // draw legend colored rectangles
    legend
      .append("rect")
      .attr("y", (d, i) => i * 30)
      .attr("height", 10)
      .attr("width", 10)
      .attr("fill", (d) => colors(d.key));

    // draw legend text
    legend
      .append("text")
      .attr("y", (d, i) => i * 30 + 9)
      .attr("x", 5 * 3)
      .text((d) => d.key);

    mainG
      .append("g")
      .call(xAxis)
      .attr("transform", `translate(0,${innerHeight})`);

    mainG.append("g").call(yAxis);

    mainG
      .append("text")
      .attr("class", "chart-title")
      .attr("y", -40)
      .attr("x", innerHeight / 2)
      .text("Suicides by Region and Gender (1985-2016)")
      .attr("font-family", "arial")
      .attr("font-size", 24);
  })
  .catch((error) => {
    console.log(error);
  });

```

I don't want to provide detailed explanation of what each line of code is doing, as comments are provided at the right places. Also, I don't want to convert this to a tutorial *(separate D3.js tutorial series is coming soon)*.

I would encourage you to experiment with the code provided and come-up with your own versions of the visualization, which is, in my opinion is a nice way of learning.

I will only take you through a few important points which will help you understand the code much easily.

This visualization is divided into 4 parts:
- Main Chart
- Chart title
- Legend
- Tooltip

**SVG** acts as a canvas which holds all the data visualization elements. The **Main chart** consists of bars for each gender which are stacked one over the other. The x-axis has **Region** and the Y-axis shows the **count**. This is **categorical data**.

We are appending chart elements to this SVG element. Also, we are grouping common elements (example: x & y axis) and adding these groups to the SVG element. The advantage of creating groups is, the chart elements are easier to maintain.

**Tooltip:** We can use a simple built-in tooltip for the chart. But for this chart, we have created a custom tooltip with a **div** element. Also, we are positioning the tooltip at a place of our choice.

Let's create a **css** file **(style.css)** which will style the chart elements. This file is only styling tooltip for now. 

## CSS file

```css
#tooltip {
  position: absolute;
  width: 200px;
  height: auto;
  padding: 10px;
  background-color: lightblue;
  -webkit-border-radius: 10px;
  -moz-border-radius: 10px;
  border-radius: 10px;
  -webkit-box-shadow: 4px 4px 10px rgba(0, 0, 0, 0.4);
  -moz-box-shadow: 4px 4px 10px rgba(0, 0, 0, 0.4);
  box-shadow: 4px 4px 10px rgba(0, 0, 0, 0.4);
  pointer-events: none;
}

#tooltip.hidden {
  display: none;
}

#tooltip p {
  margin: 0;
  font-family: sans-serif;
  font-size: 16px;
  line-height: 16px;
}

```

Here is the [final visualization](https://codepen.io/ksp585/full/qBWydYq).

### Analysis

- From the chart it is clear that suicide rate among **Males** is high compared to **Females**.
- Region wise, **Europe** tops the list with most suicides for the period 1985-2016.

This chart only gives us a preliminary view of the data which answers our question we started off with, which region has highest suicide rates and how is this data distributed by gender.

There is scope for more analysis. You can find a lot of other data points connected to this dataset in this [file](https://raw.githubusercontent.com/learning-monk/DataVisualizationProjects_JS/master/DataViz_with_D3js/Stacked%20bar%20chart%20-%20suicides%20by%20region/suicides_full_data.csv).

Feel free to conduct your own analysis and share your work.
