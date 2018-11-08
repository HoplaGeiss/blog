---
title: "Angular 5 / d3.js - Force Directed Chart"
description: "Create a force directed chart with angular5 and d3.js"
tags:
  - angular5
  - d3.js
date: "2017-11-20"
featured: false
slug: angular5-d3js-force-directed-chart
image: /images/force-directed-chart.png
---

In this post you will learn how to create a dynamic and resizeable force directed chart with angular5 and d3.js.

Take a look at the demo:
[http://www.muller.tech/hopla-force-directed-chart/](http://www.muller.tech/hopla-force-directed-chart/)

If you are in a hurry, you can checkout the source code here: [https://github.com/HoplaGeiss/hopla-force-directed-chart](https://github.com/HoplaGeiss/hopla-force-directed-chart)

Now, let's dive in!

### Component skeleton

```typescript
import { Component, OnInit, OnChanges, ViewChild, ElementRef, Input, AfterViewChecked } from '@angular/core';
import * as d3 from 'd3';
import * as _ from 'underscore';

@Component({
  selector: 'app-force-directed-chart',
  styleUrls: ['./force-directed-chart.component.scss'],
  template: `
    <div class='force-directed-chart' #forceDirectedChartContainer>
      <div class='tooltip'>
        <div fxLayout='row' fxLayoutAlign='start center' class='header'>
          <img src='assets/Icon-Sensors.svg' alt='Sensors Icon'>
          <div fxLayout='column' fxLayoutAlign='center start'>
            <div class='title'>{{node?.vendor}}</div>
            <div>{{node?.model}}</div>
          </div>
        </div>
        <div fxLayout='column' fxLayoutAlign='center start' class='body'>
          <div class='subtitle'>Name</div>
          <div>{{node?.name}}</div>
          <div class='subtitle'>IP</div>
          <div>{{node?.address}}</div>
          <div class='subtitle'>Protocol</div>
          <div>{{node?.protocol}}</div>
        </div>
      </div>
    </div>
  `
})

export class ForceDirectedChartComponent implements OnInit, OnChanges, AfterViewChecked {
  @ViewChild('forceDirectedChartContainer') private chartContainer: ElementRef;
  @Input() data: Array<any>;

  node: any;

  hostElement: any;
  tooltip: any;

  linkElements: any;
  linkGroup: any;
  nodeElements: any;
  nodeGroup: any;
  imageGroup: any;
  textElements: any;
  textGroup: any;
  imageElements: any;
  links: any;

  margin: any;
  width: any;
  height: any;
  hostElementHeight: number;

  color: any;

  svg: any;
  simulation: any;
  sensors: any;

  smallCircleRadius = 6;
  largeCircleRadius = 50;

  status: string;

  constructor(
    private elementRef: ElementRef
  ) {}
}
```

Not so much going on here. In the template we define the containers for the svg and for the tooltip. In the component itself we just define loads and loads of variables.


### Life cycle methods


```typescript
ngOnInit() {
  this.hostElement = this.chartContainer.nativeElement;
}

ngOnChanges() {
  if (this.data.length > 0 && this.svg) {
    this.sensors = _.sortBy(this.data, 'familyType');
    this.runSimulation();
  }
}

() {
  if (this.hostElement && this.hostElement.offsetHeight !== 0 && this.hostElement.offsetHeight !== this.hostElementHeight) {
    this.hostElementHeight = this.hostElement.offsetHeight;
    d3.select('.svg').remove();
    this.createChart();
    if (this.data.length > 0) {
      this.sensors = _.sortBy(this.data, 'familyType');
      this.runSimulation();
    }
  }
}
```

Let's go through this life cycle methods one by one.

- ngOnInit: we just define the `hostElement` variable.
- ngOnChanges: we update the chart with when new nata comes in.
- ngAfterViewChecked: If the screen is resized, we delete the previous svg and create a new one.

### Create the chart

```typescript
createChart = () => {
  // Width & height
  this.margin = { top: 0, right: 0, bottom: 0, left: 0};

  this.width = this.hostElement.offsetWidth - this.margin.left - this.margin.right;
  this.height = this.hostElement.offsetHeight - this.margin.top - this.margin.bottom;

  // Color
  this.color = d3.scaleOrdinal(d3.schemeCategory20);

  this.svg = d3.select(this.hostElement)
    .append('svg')
    .attr('width', this.hostElement.offsetWidth)
    .attr('height', this.hostElement.offsetHeight)
    .append('g')
    .attr('transform', 'translate(' + this.margin.left + ',' + this.margin.top + ')');

  this.linkGroup = this.svg.append('g').attr('class', 'links');
  this.nodeGroup = this.svg.append('g').attr('class', 'nodes');
  this.textGroup = this.svg.append('g').attr('class', 'labels');
  this.imageGroup = this.svg.append('g').attr('class', 'images');

  this.tooltip =  this.elementRef.nativeElement.querySelector('.tooltip');
}
```

This method creates the svg and instantiates the different groups and the tooltip.

### Run the simulation

```typescript

runSimulation = () => {
  this.simulation = d3.forceSimulation()
    .force('link', d3.forceLink().id((node: any) => node.id).distance(30))
    .force('collide', d3.forceCollide<any>( d => d.r + 8 ).iterations(16))
    .force('charge', d3.forceManyBody()
      .strength(-100)
      .distanceMax(200)
      .distanceMin(60)
    )
    .force('center', d3.forceCenter(this.width / 2, this.height / 2));

  this.render();

  this.simulation.nodes(this.sensors).on('tick', () => {
    this.nodeElements
      .attr('cx', node => this.placeElementX(node) )
      .attr('cy', node => this.placeElementY(node))
      .attr('fill', node => this.color(node.familyType));

    this.linkElements
      .attr('x1', node => this.placeElementX(node.source))
      .attr('y1', node => this.placeElementY(node.source))
      .attr('x2', node => this.placeElementX(node.target))
      .attr('y2', node => this.placeElementY(node.target));

    this.textElements
      .attr('x', groups => this.placeElementX(groups[groups.length - 1]))
      .attr('y', groups => this.placeElementY(groups[groups.length - 1]))
      .text(groups => groups[0].familyType);
  });

  this.simulation.force('link').links(this.links);
}
```

The simulation defines all sorts of physic parameters like how strongly the node are attracted or repulsed from each other, the minimum and maximum distance between nodes and so on.

On tick, we define the new location of the node. To do this we are using `placeElementX` and `placeElementY` because we want to keep the node within our container.

### Render

```typescript
render = () => {
  this.links = this.createLinks();

  this.linkElements = this.linkGroup.selectAll('line')
    .data(this.links, link => link.target.id + link.source.id);

  this.linkElements.exit().remove();

  const linkEnter = this.linkElements
    .enter().append('line')
    .attr('stroke-width', 1)
    .attr('stroke', '#d2d7d3');

  this.linkElements = linkEnter.merge(this.linkElements);

	// nodes
  this.nodeElements = this.nodeGroup.selectAll('circle')
    .data(this.sensors, node => node.id);

  this.nodeElements.exit().remove();

  const nodeEnter = this.nodeElements
    .enter()
    .append('circle')
    .attr('r', this.smallCircleRadius)
    .attr('fill', node => this.color(node.familyType))
    .on('mouseover', node => this.mouseover(node))
    .on('mouseout', node => this.mouseout())
    .call(d3.drag()
      .on('start', this.dragstarted)
      .on('drag', this.dragged)
      .on('end', this.dragended));

  this.nodeElements = nodeEnter.merge(this.nodeElements);

  // text labels
  const groups = _.toArray(_.groupBy(this.sensors.slice(), 'familyType'));

  this.textElements = this.textGroup.selectAll('text')
    .data(groups, group => group);

  this.textElements.exit().remove();

  const textEnter = this.textElements
    .enter()
    .append('text')
    .attr('dx', 15)
    .attr('dy', 4)
    .attr('pointer-events', 'none')
    .attr('font-size', 14)
    .attr('text-anchor', 'start')
    .attr('fill', group => this.color(group[0].familyType));

  this.textElements = textEnter.merge(this.textElements);
}
```

Here we crunch the data and define the enter, update and exit behaviour for the nodes, the labels and the links.

### Other methods

```typescript
createLinks = () => {
  const groups = _.groupBy(this.sensors.slice(), 'familyType');

  const computedLinks = [];
  Object.keys(groups).map((key, i) => {
    const familyTypeSensors = groups[key].sort();
    for (let j = 0; j < familyTypeSensors.length; j++) {
      if (j > 0) computedLinks.push({
        source: familyTypeSensors[j],
        target: familyTypeSensors[j - 1]
      });
    }
  });

  return computedLinks;
}

dragstarted = (d) => {
  if (!d3.event.active) this.simulation.alphaTarget(0.3).restart();
  d.fx = d.x;
  d.fy = d.y;
}

dragged = (d) => {
  d.fx = d3.event.x;
  d.fy = d3.event.y;
}

dragended = (d) => {
  if (!d3.event.active) this.simulation.alphaTarget(0);
  d.fx = null;
  d.fy = null;
}

mouseover = (node) => {
  // placement
  this.tooltip.style.visibility = 'visible';
  this.tooltip.style.opacity = 0.9;
  this.tooltip.style.top = d3.event.pageY + 'px';
  this.tooltip.style.left = d3.event.pageX + 'px';

  // Values
  this.node = node;
  this.status = this.findStatus(node);
}

findStatus = (node): string => {
  let status = '';
  if (!node.activated) status = 'Not activated';
  if (node.activated && node.online)  status = 'Online';
  if (node.activated && !node.online) status = 'Offline';
  return status;
}

mouseout = () => {
  this.tooltip.style.visibility = 'hidden';
  this.tooltip.style.opacity = 0;
}

placeElementY = (node): number => {
  const radius = this.smallCircleRadius;

  if (node.y < radius) return radius;
  if (node.y > this.height - radius) return this.height - radius;
  return node.y;
}

placeElementX = (node): number => {
  const radius = this.smallCircleRadius;

  if (node.x < radius) return radius;
  if (node.x > this.width - radius) return this.width - radius;
  return node.x;
}
```

### Data service
```typescript
import { Injectable, OnInit } from '@angular/core';

@Injectable()
export class ForceDirectedDataService {
  sensors = [];
  families = ['Temperature', 'Luminosity', 'Noise', 'Humidity', 'Dust'];
  maxSensors = 30;

  createSensors = (num: number) => {
    const max = Math.floor(Math.random() * num);
    // create sensors
    const results = [];
    for (let i = 0; i < max; i++) {
      const result = {
        id: i,
        familyType: this.families[Math.floor(Math.random() * this.families.length)],
        address: '192.168.0.1',
        name: 'MAN1-PMC-Temperature-Sensor-01',
        vendor: 'Veris',
        model: 'TED00',
        protocol: '3-Wire'
      };
      results.push(result);
    }
    return results;
  }
}
```

### Styling

```scss
.force-directed-chart {
  width: 100%;
  height: 400px !important;
}

.tooltip{
  position: absolute;
  background: #f5f6f5;
  box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);

  opacity: 0;
  transition: visibility 0s, opacity 0.5s linear;
  visibility: hidden;
  pointer-events: none;

  .header{
    color: $primary;
    padding: 20px;
    border-bottom: 1px solid $border-color;
    .title{
      font-weight: bold;
      font-size: 1.2em;
    }

    img{
      margin-right: 20px;
    }
  }

  .body{
    .subtitle{
      font-weight: bold;
      color: $primary;
      filter: brightness(90%);
    }
    padding: 20px;
  }
}
```

Happy plotting! Have a look at the source code here: [https://github.com/HoplaGeiss/hopla-force-directed-chart](https://github.com/HoplaGeiss/hopla-force-directed-chart)
