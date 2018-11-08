---
title: "Angular 5 / d3.js - Pie Chart"
description: "Create a dynamic and resizable pie chart with angular5 and d3.js"
tags:
  - angular5
  - d3.js
date: "2017-11-13"
featured: false
slug: angular5-d3js-pie-chart
image: /images/pie-chart.png
---

In this post you will learn how to create a dynamic and resizable pie chart with angular5 and d3.js.
Pie charts are very useful to present a number of data, however I couldn't find any example online working with angular2+ and d3 v4, so I decided to implement my own from scratch.

Take a look at the demo:
[http://www.muller.tech/hopla-pie-chart/](http://www.muller.tech/hopla-pie-chart/)

If you are in a hurry, you can checkout the source code here: [https://github.com/HoplaGeiss/hopla-pie-chart](https://github.com/HoplaGeiss/hopla-pie-chart)

Now, let's dive in!

### Component skeleton

```typescript
import { Component, OnInit, OnChanges, ViewChild, ElementRef, Input } from '@angular/core';
import * as d3 from 'd3';
import { SumPipe } from '../_pipes/sum.pipe';
import * as _ from 'underscore';

@Component({
    selector: 'app-pie-chart',
    styleUrls: ['./pie-chart.component.scss'],
    template: `
      <div class='wrapper'>
        <div class='pie-chart' #containerPieChart></div>

        <div *ngIf='labels'class='legend' fxLayout='row' fxLayoutAlign='center center'>
          <div *ngFor='let label of labels; let i = index' fxLayout='row' fxLayoutAlign='start center' class='item'>
            <div class='circle' [ngStyle]='{"background": colourSlices[i]}'></div>
            <div>{{label}}</div>
          </div>
        </div>

        <div class='tooltip drilldownData'>
          <div class='header' *ngIf='selectedSlice'>
            <div fxLayout='row' fxLayoutAlign='start center'>
              <img [src]='"assets/Icon-" + selectedSlice.familyType + ".svg"' alt='Database Icon'>
              <div fxLayout='column' fxLayoutAlign='center start'>
                <div class='title'>{{selectedSlice.familyType}}</div>
                <div>Queries</div>
              </div>
            </div>
          </div>
          <div class='body' fxLayout='row' fxLayoutWrap *ngIf='selectedSlice'>
            <div *ngFor='let item of selectedSlice.types' class='item' fxFlex="50%">
              <div class='label'>{{item.type}}</div>
              <div class='value'>{{item.amount}}</div>
            </div>
          </div>
        </div>
      </div>
    `
})

export class PieChartComponent implements OnInit, OnChanges {
  @ViewChild('containerPieChart') chartContainer: ElementRef;
  @Input() data: any;
  @Input() colours: Array<string>;

  hostElement: any;
  svg: any;
  radius: number;
  innerRadius: number;
  outerRadius: number;
  htmlElement: HTMLElement;
  arcGenerator: any;
  arcHover: any;
  pieGenerator: any;
  path: any;
  values: Array<number>;
  labels: Array<string>;
  tooltip: any;
  centralLabel: any;
  pieColours: any;
  slices: Array<any>;
  selectedSlice: any;
  colourSlices: Array<string>;
  arc: any;
  arcEnter: any;

  constructor(
    private elRef: ElementRef
  ) {}
}
```

The most important in this piece of code is the template. You can see it is split in three parts, the chart, the labels and the tooltip.

It is also worth noticing the inputs of this component. `data` is an array of object we want to plot and `colours` is an array of string that we will use for colouring the slices of the pie chart.

### Data Service

```typescript
import { Injectable } from '@angular/core';

@Injectable()
export class PieDataService {
  generateData = (num: number) => {
    const operations = [];
    const labels = ['Devices', 'Database', 'API'];
    const types = ['SnmpV1', 'SnmpV2c', 'SnmpV3', 'HttpApi', 'HttpBasic', 'SshBasic', 'SshRsa', 'Wmi', 'Sql', 'MongoDb'];

    for (let i = 0; i < Math.floor(1 + Math.random() * num); i++) {
      const operation = {
        id: i,
        familyType: labels[Math.floor(Math.random() * labels.length)],
        name: 'MAN1-APC-01-Get-Power-Consumption',
        type: types[Math.floor(Math.random() * types.length)]
      };

      operations.push(operation);
    }

    return operations;
  }
}
```

In my example I am going to plot a pie chart of operations.

### Create chart

```typescript
createChart = () => {
  // chart configuration
  this.hostElement = this.chartContainer.nativeElement;

  this.radius = Math.min(this.hostElement.offsetWidth, this.hostElement.offsetHeight) / 2;
  const innerRadius = this.radius - 80;
  const outerRadius = this.radius - 15;
  const hoverRadius = this.radius - 5;
  this.pieColours = this.colours ? d3.scaleOrdinal().range(this.colours) : d3.scaleOrdinal(d3.schemeCategory20c);
  this.tooltip = this.elRef.nativeElement.querySelector('.tooltip');

  // create a pie generator and tell it where to get numeric values from and whether sorting is needed or not
  // this is just a function that will be called to obtain data prior binding that data to elements of the chart
  this.pieGenerator = d3.pie().sort(null).value((d: number) => d)([0, 0, 0]);

  // create an arc generator and configure it
  // this is just a function that will be called to obtain data prior binding that data to arc elements of the chart
  this.arcGenerator = d3.arc()
    .innerRadius(innerRadius)
    .outerRadius(outerRadius);

  this.arcHover = d3.arc()
    .innerRadius(innerRadius)
    .outerRadius(hoverRadius);

  // create svg element, configure dimentions and centre and add to DOM
  this.svg = d3.select(this.hostElement).append('svg')
    .attr('viewBox', '0, 0, ' + this.hostElement.offsetWidth + ', ' + this.hostElement.offsetHeight)
    .append('g')
    .attr('transform', `translate(${this.hostElement.offsetWidth / 2}, ${this.hostElement.offsetHeight / 2})`);
}
```

In this `createChart` method we mostly define variables and create the svg.


### updateChart

```typescript
updateChart = (firstRun: boolean) => {
  const vm = this;

  this.slices = this.updateSlices(this.data);
  this.labels = this.slices.map(slice => slice.familyType);
  this.colourSlices = this.slices.map(slice => this.pieColours(slice.familyType));

  this.values = firstRun ? [0, 0, 0] : _.toArray(this.slices).map(slice => slice.amount);

  this.pieGenerator = d3.pie().sort(null).value((d: number) => d)(this.values);

  const arc = this.svg.selectAll('.arc')
    .data(this.pieGenerator);

  arc.exit().remove();

  const arcEnter = arc.enter().append('g')
    .attr('class', 'arc');

  arcEnter.append('path')
    .attr('d', this.arcGenerator)
    .each((values) => firstRun ? values.storedValues = values : null)
    .on('mouseover', this.mouseover)
    .on('mouseout', this.mouseout);

  // configure a transition to play on d elements of a path
  // whenever new values are passed in, the values and the previously stored values will be used
  // to compute the transition using interpolation
  d3.select(this.hostElement).selectAll('path')
    .data(this.pieGenerator)
    .attr('fill', (datum, index) => this.pieColours(this.labels[index]))
    .attr('d', this.arcGenerator)
    .transition()
    .duration(750)
    .attrTween('d', function(newValues, i){
      return vm.arcTween(newValues, i, this);
    });
}
```

In the `updateChart` we do a number of things:

- Call `updateSlices` to map the data in a way d3.js can use it.
- Extract all the labels in `labels`.
- Assign a colour for each slices.
- Define the behaviour on enter. (`mouseover` and `mouseout`)
- Define the behaviour on update. (re-calculate the angles of the slices )
- Defined the behaviour on exit. (remove)


### Other methods

```typescript
arcTween(newValues, i, slice) {
  const interpolation = d3.interpolate(slice.storedValues, newValues);
  slice.storedValues = interpolation(0);

  return (t) => {
    return this.arcGenerator(interpolation(t));
  };
}

mouseover = (d, i) => {
  this.selectedSlice = this.slices[i];

  d3.select(d3.event.currentTarget).transition()
    .duration(200)
    .attr('d', this.arcHover);

  this.svg.append('text')
    .attr('dy', '-10px')
    .style('text-anchor', 'middle')
    .attr('class', 'label')
    .attr('fill', '#57a1c6')
    .text(this.labels[i]);

  this.svg.append('text')
    .attr('dy', '20px')
    .style('text-anchor', 'middle')
    .attr('class', 'percent')
    .attr('fill', '#57a1c6')
    .text(this.toPercent(this.values[i], new SumPipe().transform(this.values)));

  // Tooltip
  this.tooltip.style.visibility = 'visible';
  this.tooltip.style.opacity = 0.9;
  this.tooltip.style.top = (d3.event.pageY) + 'px';
  this.tooltip.style.left = (d3.event.pageX - 100 ) + 'px';
}

mouseout = () => {
  this.svg.select('.label').remove();
  this.svg.select('.percent').remove();

  d3.select(d3.event.currentTarget).transition()
   .duration(100)
   .attr('d', this.arcGenerator);

  this.tooltip.style.visibility = 'hidden';
  this.tooltip.style.opacity = 0;
}

toPercent = (a: number, b: number): string => {
  return Math.round( a / b * 100) + '%';
}

updateSlices = (newData: Array<any>): Array<any> => {
  const queriesByFamilyTypes = _.groupBy(_.sortBy(newData, 'familyType'), 'familyType');
  const results = [];

  Object.keys(queriesByFamilyTypes).map((familyType) => {
    results.push({
      familyType: familyType,
      amount: queriesByFamilyTypes[familyType].length,
      types: []
    });
  });

  results.map(result => {
    const queries = newData.filter(query => query.familyType === result.familyType);
    const queriesByTypes = _.groupBy(_.sortBy(queries, 'type'), 'type');

    Object.keys(queriesByTypes).map((type) => {
      result.types.push({
        type: type,
        amount: queriesByTypes[type].length,
      });
    });
  });

  return results;
}
```

### OnInit & OnChanges

```typescript
ngOnInit() {
  // create chart and render
  this.createChart();

  // Initial update
  this.updateChart(true);

  // For animation purpose we load the real value after a second
  setTimeout(() => this.updateChart(false), 50);
}

ngOnChanges() {
  // update chart on data input value change
  if (this.svg) this.updateChart(false);
}
```

Our `OnInit` method is a little peculiar, we first call update chart to initialize it and then call it a second time a little bit later to give it its real value. That's is done in order to have a nice animation.

### Conclusion

I realise this post contains a lot more code and therefore less explanation than I usually like to have, but the easiest might be to checkout my code and tweak it to see how it works. Here is the link to the source code: [https://github.com/HoplaGeiss/hopla-pie-chart](https://github.com/HoplaGeiss/hopla-pie-chart)
