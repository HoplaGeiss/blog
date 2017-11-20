---
title: "Angular 5 / d3.js - Bubble Chart"
description: "Create a dynamic and resizable bubble chart with angular5 and d3.js"
tags:
  - angular5
  - d3.js
date: "2017-11-20"
featured: true
slug: angular5-d3js-bubble-chart
image: /images/bubble-chart.png
---

In this post you will learn how to create a dynamic and resizable bubble chart with angular5 and d3.js.

Take a look at the demo:
[http://www.muller.tech/hopla-bubble-chart/](http://www.muller.tech/hopla-bubble-chart/)

If you are in a hurry, you can checkout the source code here: [https://github.com/HoplaGeiss/hopla-bubble-chart](https://github.com/HoplaGeiss/hopla-bubble-chart)

Now, let's dive in!

### Component skeleton

```typescript
import { Component, ElementRef, Input, ViewChild, OnInit, OnChanges, AfterViewChecked } from '@angular/core';
import * as d3 from 'd3';
import * as _ from 'underscore';

@Component({
  selector: 'app-bubble-chart',
  styleUrls: ['./bubble-chart.component.scss'],
  template: `
    <div class='bubble-chart' #bubbleChartContainer>
      <div class='tooltip'>
        <div fxLayout='row' fxLayoutAlign='start center'>
          <img *ngIf='type' [src]="'assets/Icon-' + type + '.svg'" [alt]="type +' Icon'">
          <div fxLayout='column' fxLayoutAlign='center center'>
            <div class='title'>{{type}}</div>
            <div>{{amount}}</div>
          </div>
        </div>
      </div>
    </div>
  `
})
export class BubbleChartComponent implements OnInit, OnChanges, AfterViewChecked {
  @ViewChild('bubbleChartContainer') private chartContainer: ElementRef;
  @Input() private data: Array<any>;

  svg: any;
  pack: any;
  transition: number;
  color: any;
  bubbles: Array<any> = [];
  tooltip: any;
  amount = 0;
  type: string;
  hostElement: any;
  hostElementWidth: number;

  constructor(
    private elementRef: ElementRef
  ) {}

  ngOnInit() {
    this.hostElement = this.chartContainer.nativeElement;
  }

  ngOnChanges() {
    if (this.data && this.data.length > 0 && this.svg) this.updateChart();
  }

  ngAfterViewChecked () {
    if (this.hostElement && this.hostElement.offsetWidth !== 0 && this.hostElement.offsetWidth !== this.hostElementWidth) {
      this.hostElementWidth = this.hostElement.offsetWidth;
      d3.select('.bubble-chart svg').remove();
      this.createChart();
      if (this.data.length > 0) this.updateChart();
    }
  }
```

This snippet encompasses the most important part of the component, let's go through it step by step.

The template:

- `bubbleChartContainer` is a container for our bubble chart's svg.
- `tooltip` is the container the tooltip that is displayed when the user hovers the bubbles.

The input:

- This components expects data as an array of objects. For instance `[{id: 0, type: 'Printer'}, {id: 1, type: 'Printer'}, {id: 2, type: 'Tablet'}]`.

Life cycle methods:

- ngOnInit: we just define the `hostElement` variable.
- ngOnChanges: we update the chart with when new nata comes in.
- ngAfterViewChecked: If the screen is resized, we delete the previous svg and create a new one.

### Create the chart

```typescript
createChart = () => {
  this.transition = 700;

  const margin = { top: 0, right: 0, bottom: 0, left: 0};
  const width = this.hostElement.offsetWidth - margin.left - margin.right;
  const height = this.hostElement.offsetHeight - margin.top - margin.bottom;

  this.svg = d3.select(this.hostElement)
    .append('svg')
    .attr('width', width + margin.left + margin.right)
    .attr('height', height + margin.top + margin.bottom);

  this.pack = d3.pack()
    .size([width, height])
    .padding(1.5);

  this.color = d3.scaleOrdinal(d3.schemeCategory20c);
  this.tooltip = this.elementRef.nativeElement.querySelector('.tooltip');
}
```

Here we:

- Create the svg.
- Define the colours.
- Define the tooltip.

### Update the chart

``` typescript
updateChart = () => {
  this.bubbles = this.updateBubbles(this.data);

  const hierarchy = d3.hierarchy({ children: this.bubbles })
    .sum((d: any) => {
      // Crazy stuff.. here in sum we path all the node, not only the leafs
      // So we need to ignore the roots
      if (d.children) return 0;
      return d.amount;
    });

  // JOIN
  const node = this.svg.selectAll('g')
    .data(this.pack(hierarchy).leaves(), d => d.data.type);

  const nodeEnter = node.enter()
    .append('g')
    .attr('transform', d => 'translate(' + d.x + ', ' + d.y + ')');

  // ENTER
  nodeEnter.append('circle')
    .style('fill', '#fff')
    .attr('r', d => 1e-6)
    .on('mouseover', this.mouseover)
    .on('mouseout', this.mouseout)
    .transition()
    .duration(700)
    .attr('r', d => d.r)
    .style('fill', d => this.color(d.data.type));

  nodeEnter.append('image')
    .attr('xlink:href', d => 'assets/Icon-' + d.data.type + '-HL.svg')
    .attr('width', 1e-6)
    .attr('height', 1e-6)
    .attr('x', d => - d.r / 2)
    .attr('y', d => - d.r / 2)
    .attr('pointer-events', 'none')
    .transition()
    .duration(this.transition)
    .attr('width', d => d.r)
    .attr('height', d => d.r);

  // EXIT
  node.select('circle').exit()
    .transition()
    .duration(700)
    .attr('r', d => 1e-6)
    .remove();

  node.select('image').exit()
    .transition()
    .duration(700)
    .attr('width', d => 1e-6)
    .attr('height', d => 1e-6)
    .remove();

  node.exit()
    .transition()
    .duration(700)
    .remove();

  // UPDATE
  node
    .transition()
    .duration(700)
    .attr('transform', d => 'translate(' + d.x + ', ' + d.y + ')')
    .style('fill', '#3a403d');

  node.select('circle').transition()
    .duration(700)
    .attr('r', d => d.r);

  node.select('image').transition()
    .duration(700)
    .attr('x', d => - d.r / 2)
    .attr('y', d => - d.r / 2)
    .attr('width', d => d.r)
    .attr('height', d => d.r);
}
```

Here we crunch the data and define the enter, update and exit behaviour for the circles and the images.

### Helpers

```typescript

  updateBubbles = (newData: Array<any>): Array<any> => {
    const dataByTypes = _.groupBy(_.sortBy(newData, 'type'), 'type');
    const results = [];
    Object.keys(dataByTypes).map((key, index) => {
      results.push({
        type: key,
        amount: dataByTypes[key].length
      });
    });

    return results;
  }

  mouseover = (d, i) => {
    // placement
    this.tooltip.style.visibility = 'visible';
    this.tooltip.style.opacity = 0.9;
    this.tooltip.style.top = d3.event.pageY + 'px';
    this.tooltip.style.left = d3.event.pageX + 'px';

    // Values
    this.amount = d.data.amount;
    this.type = d.data.type;
  }

  mouseout = () => {
    this.tooltip.style.visibility = 'hidden';
    this.tooltip.style.opacity = 0;
  }

```


### Data Service

```typescript
import { Injectable } from '@angular/core';

@Injectable()
export class BubbleChartDataService {
  generateData = (num: number) => {
    const results = [];
    const devices = ['Printer', 'Router', 'Tablet', 'Workstation'];

    for (let i = 0; i < num; i++) {
      const result = {
        id: i,
        type: devices[Math.floor(Math.random() * devices.length)]
      };

      results.push(result);
    }

    return results;
  }
}
```

### Styling

```scss
.bubble-chart{
  width: 100%;
  height: 100% !important;
}

.tooltip{
  position: absolute;
  background: #f5f6f5;
  box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
  padding: 20px;
  color: #57a1c6;
  opacity: 0;
  transition: visibility 0s, opacity 0.5s linear;
  visibility: hidden;
  pointer-events: none;

  img{
    margin-right: 20px;
  }
}
```

That's all for this post.. Have a look here for the source code:
[https://github.com/HoplaGeiss/hopla-bubble-chart](https://github.com/HoplaGeiss/hopla-bubble-chart)
