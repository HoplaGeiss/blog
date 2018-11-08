---
title: "Angular 5 / d3.js - Scatter Chart"
description: "Create a scatter chart with angular5 and d3.js"
tags:
  - angular5
  - d3.js
date: "2017-11-20"
featured: false
slug: angular5-d3js-scatter-chart
image: /images/scatter-chart.png
---

In this post you will learn how to create a scatter chart with angular5 and d3.js.

Take a look at the demo:
[http://www.muller.tech/hopla-scatter-chart/](http://www.muller.tech/hopla-scatter-chart/)

If you are in a hurry, you can checkout the source code here: [https://github.com/HoplaGeiss/hopla-scatter-chart](https://github.com/HoplaGeiss/hopla-scatter-chart)

Now, let's dive in!

### Component skeleton

```typescript
import { Component, ElementRef, ViewChild, OnInit, Input } from '@angular/core';
import * as d3 from 'd3';

@Component({
  selector: 'app-scatter-chart',
  styleUrls: ['./scatter-chart.component.scss'],
  template: `
    <div class='scatter-chart' #scatterChartContainer>
      <div class='tooltip'>
        <span class='highlight'>Device discovered:</span> {{node?.value}}
      </div>
    </div>
  `
})
export class ScatterChartComponent implements OnInit {
  @ViewChild('scatterChartContainer') private chartContainer: ElementRef;
  @Input() private data: Array<any>;
  @Input() monthsAgo: number;

  hostElement: any;

  margin: any;
  width: number;
  height: number;
  groupWidth: number;
  groupHeight: number;

  dotColours: any;
  labelColours: any;

  formatTime: any;

  minDt: any;
  maxDt: any;

  xAxisScale: any;
  xAxis: any;
  gX: any;

  groupCircles: any;
  tooltip: any;
  circles: any;
  outerCircle: any;
  innerCircle: any;

  zoom: any;

  svg: any;
  view: any;

  node: any;

  constructor(
    private elementRef: ElementRef
  ) {}
}

```

Not so much going on here. In the template we define the containers for the svg and for the tooltip. In the component itself we just define loads and loads of variables.


### Initialization

```typescript
ngOnInit() {
  this.hostElement = this.chartContainer.nativeElement;
  this.tooltip = this.elementRef.nativeElement.querySelector('.tooltip');

  // Width & Height
  this.margin = { top: 0, right: 0, bottom: 30, left: 0};

  this.width = this.hostElement.offsetWidth - this.margin.left - this.margin.right;
  this.height = this.hostElement.offsetHeight - this.margin.top - this.margin.bottom;

  this.groupWidth =  200;

  this.groupHeight = this.height / this.data.length;

  // Colours
  this.dotColours = d3.scaleOrdinal().range(['#3FBBA2', '#904FAD']);
  this.labelColours = d3.scaleOrdinal().range(['#049372', '#674172']);

  // Formating
  this.formatTime = d3.timeFormat('%e %B');

  this.minDt = this.addMonths(new Date(), -this.monthsAgo);
  this.maxDt = new Date();

  this.render();
}
```

In this init method, we already have more stuff going on! We instantiate the host element, the tooltip, the dot colours, the label colours. We also define the time formats and the beginning and the end of the x scale.

### Render the svg

```typescript
render = () => {
    this.xAxisScale = d3.scaleTime()
      .domain([this.minDt, this.maxDt]) // Defines min and max of the x axis
      .range([this.groupWidth, this.width]);  // Defines where the x axus label begin and end

    this.xAxis = d3.axisBottom(this.xAxisScale)
      .tickSize(-this.height - 10); // Defines the height of the vertical ligne above the x-axis

    this.zoom = d3.zoom()
      .on('zoom', this.zoomFn);

    this.svg = d3.select(this.hostElement)
      .append('svg')
      .attr('width', this.width + this.margin.left + this.margin.right)
      .attr('height', this.height + this.margin.top + this.margin.bottom)
      .append('g')
      .attr('transform', 'translate(' + this.margin.left + ',' + this.margin.top + ')')
      .call(this.zoom);

    this.view = this.svg
      .append('defs')
      .append('clipPath')
      .attr('id', 'chart-content')
      .append('rect')
      .attr('x', this.groupWidth)
      .attr('y', 0)
      .attr('height', this.height)
      .attr('width', this.width - this.groupWidth);

    // Rectangle with data
    this.svg.append('rect')
      .attr('class', 'chart-bounds')
      .attr('x', this.groupWidth)
      .attr('y', 0)
      .attr('height', this.height)
      .attr('width', this.width - this.groupWidth);

    // Rectangle with groups
    this.svg.append('rect')
      .attr('class', 'group-bounds')
      .attr('x', 0)
      .attr('y', 0)
      .attr('height', this.height)
      .attr('width', this.groupWidth);

    // Dates of the x axis
    this.gX = this.svg.append('g')
      .attr('class', 'x axis')
      .attr('transform', 'translate(0,' + (this.height + 10) + ')')
      .call(this.xAxis);

    this.drawBorders();
    this.createHorizontalSectionLines();
    this.createSectionLabels();
    this.createCircles();
  }
```

Here is where the magic happens. We create and svg, we put in place elements to allow panning and zooming, we define the zones where the labels and the data will be displayed, we put in place the x-axis and to finish we call the other methods to draw the borders, create the section lines, create the section labels and finally create the circles.

### Other methods

```typescript
createSectionLabels = () => {
  this.svg.selectAll('.group-label')
    .data(this.data)
    .enter()
    .append('text')
    .attr('class', 'group-label')
    .attr('x', 0)
    .attr('y', (d, i) =>  this.groupHeight * i + this.groupHeight / 2 + 5.5)
    .attr('dx', '0.5em').text(d => d.label)
    .attr('fill', d => this.labelColours(d.label));
}

createHorizontalSectionLines = () => {
  this.svg.selectAll('.group-section')
    .data(this.data)
    .enter()
    .append('line')
    .attr('class', 'group-section')
    .attr('x1', 0)
    .attr('x2', this.width)
    .attr('y1',  (d, i) => this.groupHeight * (i + 1))
    .attr('y2', (d, i) => this.groupHeight * (i + 1));
}

createCircles = () => {
  // Create groups of items with different colours
  this.groupCircles = this.svg.selectAll('.group-dot-item')
    .data(this.data)
    .enter()
    .append('g')
    .attr('clip-path', 'url(#chart-content)')
    .attr('class', 'item')
    .attr('transform', (d, i) => 'translate(0, ' + this.groupHeight * i + ')')
    .attr('fill', d => this.dotColours(d.label))
    .selectAll('.dot')
    .data(d => d.data).enter();

  // Group the outer and inner circle together
  this.circles = this.groupCircles.append('g')
    .attr('class', 'circle')
    .on('mouseover', node => this.mouseOverFn(node))
    .on('mouseout', node => this.mouseOutFn());

  this.outerCircle = this.circles.append('circle')
    .attr('class', 'dot')
    .attr('cx', d => this.xAxisScale(d.time))
    .attr('cy', this.groupHeight / 2)
    .attr('r', 7);

  this.innerCircle = this.circles.append('circle')
    .attr('class', 'inner-circle')
    .attr('cx', d => this.xAxisScale(d.time))
    .attr('cy', this.groupHeight / 2)
    .attr('r', 3)
    .attr('fill', 'white');
}

mouseOverFn = (node) => {
  this.node = node;

  this.tooltip.style.visibility = 'visible';
  this.tooltip.style.opacity = 0.9;
  this.tooltip.style.top = (d3.event.pageY - 35) + 'px';
  this.tooltip.style.left = (d3.event.pageX - 35) + 'px';
}

mouseOutFn = () => {
  this.tooltip.style.visibility = 'hidden';
  this.tooltip.style.opacity = 0;
}

zoomFn = () => {
  // Transform the xAxis
  this.gX.call(this.xAxis.scale(d3.event.transform.rescaleX(this.xAxisScale)));

  // Translate circle on the X axis
  const newXScale = d3.event.transform.rescaleX(this.xAxisScale);
  this.innerCircle.attr('cx', d => newXScale(d.time));
  this.outerCircle.attr('cx', d => newXScale(d.time));
}

addMonths = (date, months) => {
  date.setMonth(date.getMonth() + months);
  return date;
}

drawBorders = () => {
  // left border
  this.svg.append('line')
    .attr('class', 'border')
    .attr('y1', 0)
    .attr('y2', this.height);

  // right border
  this.svg.append('line')
    .attr('class', 'border')
    .attr('x1', this.width)
    .attr('x2', this.width)
    .attr('y1', 0)
    .attr('y2', this.height);

  // top border
  this.svg.append('line')
    .attr('class', 'border')
    .attr('x1', 0)
    .attr('x2', this.width);

  // Line between groups label on the left and plot ( make it thicker)
  this.svg.append('line')
    .attr('x1', this.groupWidth)
    .attr('x2', this.groupWidth)
    .attr('y1', 0)
    .attr('y2', this.height);
}
```

I let you go through this method, they are fairly straightforward.

### Data service

```typescript
import { Injectable } from '@angular/core';

@Injectable()
export class ScatterDataService {
  generateData = (num: number) => {
    const results = [];
    const booleans = [true, false];

    for (let i = 0; i < Math.floor(1 + Math.random() * num); i++) {
      const result = {
        id: i,
        singleRun: booleans[Math.floor(Math.random() * 2)],
        startDateUtc: new Date(2017, Math.floor(Math.random() * 12), Math.floor(Math.random() * 25)),
        totalDevicesFound: Math.floor(1 + Math.random() * 10)
      };

      results.push(result);
    }

    return results;
  }

  parseData = (discoveries: any) => {
    const single = [];
    const periodic = [];

    discoveries.map(discovery => {
      if (discovery.singleRun) {
        single.push({
          time: discovery.startDateUtc,
          value: discovery.totalDevicesFound
        });
      } else {
        periodic.push({
          time: discovery.startDateUtc,
          value: discovery.totalDevicesFound
        });
      }
    });

    return [
      { label: 'Periodic', data: periodic },
      { label: 'Single', data: single }
    ];
  }
}
```

I split the service in two methods, one to generate raw data and one to parse it in a format optimized for the use in our scatter chart. As you can see, the data is placed within the data attribute of one of our two objects in function of whether they are periodic and single.

Happy plotting! Have a look at the source code here: [https://github.com/HoplaGeiss/hopla-scatter-chart](https://github.com/HoplaGeiss/hopla-scatter-chart)
