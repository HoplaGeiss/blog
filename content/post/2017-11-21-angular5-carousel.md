---
title: "Angular 5 carousel"
description: "Create a carousel using angular 5"
tags:
  - angular5
date: "2017-11-21"
featured: true
slug: angular5-carousel
image: /images/carousel.png
---

In this post you will learn how to create a carousel with angular5.
Requirements for our carousel

- Take as an input an array of object.
- Have a left and a right arrow for navigation.
- Distribute batches of items evenly across the width of the screen.
- Update of screen resize.

Take a look at the demo:
[http://www.muller.tech/hopla-carousel/](http://www.muller.tech/hopla-carousel/)

If you are in a hurry, you can checkout the source code here: [https://github.com/HoplaGeiss/hopla-carousel](https://github.com/HoplaGeiss/hopla-carousel)

Let's dive in!

### Template

```typescript
@Component({
  selector: 'app-carousel',
  styleUrls: ['./carousel.component.scss'],
  template: `
    <div class='wrapper'>
      <div class='carousel-wrapper'>
        <div class='carousel' fxLayout='row' fxLayoutAlign='center stretch'>
          <ul *ngFor='let items of batches' fxLayout='row' fxLayoutAlign='space-evenly stretch'>
            <li *ngFor='let item of items' fxLayout='column' fxLayoutAlign='center center'>
              <img src='assets/Icon-Database.svg' alt='Widget Icon'>
              <div class='text'>{{item.title}}</div>
            </li>
          </ul>
        </div>
      </div>

      <div *ngIf='batches.length > 1' class='arrow-left' (click)='currentItem !== 0 ? slide(1) : null'
        [ngClass]='{"disabled": currentItem === 0}'>
        <i class='material-icons'>keyboard_arrow_left</i>
      </div>

      <div *ngIf='batches.length > 1' class='arrow-right' (click)='currentItem !== batches.length - 1 ? slide(-1) : null'
        [ngClass]='{"disabled": currentItem === batches.length - 1}'>
        <i class='material-icons'>keyboard_arrow_right</i>
      </div>
    </div>
  `
})
```

Our template is clearly separated in there parts:
- The carousel with an icon and a label for each item
- The left arrow which is disabled when the user looks at the first batch of items.
- The right arrow which is disabled when the user looks at the last batch of items.


### Component with life cycle methods

```typescript
export class CarouselComponent implements OnInit, AfterViewChecked {
  @Input() items: Array<object>;
  currentItem = 0;
  carousel: any;
  position = 0;
  increment: any;
  translation = 0;
  elementsPerSlide: number;
  batches: Array<Array<object>> = [];
  carouselWidth: number;
  carouselWrapper: any;
  carouselWrapperWidth: number;

  constructor (
    private elementRef: ElementRef,
    private changeDetectorRef: ChangeDetectorRef
  ) {}

  ngOnInit() {
    this.carousel = this.elementRef.nativeElement.querySelector('.carousel');
    this.carouselWrapper = this.elementRef.nativeElement.querySelector('.carousel-wrapper');
  }

  // On size changes, recalculate the bacthes
  ngAfterViewChecked() {
    if (this.carouselWrapper.offsetWidth !== 0 && this.carouselWrapper.offsetWidth !== this.carouselWrapperWidth) {
      this.carouselWrapperWidth = this.carouselWrapper.offsetWidth;

      this.renderBatches();
      this.setBatchSize();
    }
  }
}
```

A couple things worth noting in this component:

- `items` is an array of object containing the items to display in the carousel.
- OnInit we set `carousel` and `carouselWrapper`.
- We use ngAfterViewChecked to resize the carousel.

### renderBatches

```typescript
renderBatches = () => {
  const itemWidth = 130;

  // we split the items in batches
  this.elementsPerSlide = Math.floor(this.carouselWrapperWidth / itemWidth);
  this.batches = this.chunk(this.items, this.elementsPerSlide);

  // The carousel is 100 * ng batches wide
  this.carousel.style.width = 100 * this.batches.length + '%';

  this.increment = (100 / this.batches.length);

  this.changeDetectorRef.detectChanges();
}
```

this methods calculates the number of elements to show per page based on the width of the carousel's wrapper and on the width of the elements. It then updates the width of the carousel, which needs to be 100 * number of batches.

### setBatchSize

```typescript
setBatchSize = () => {
  const ulElements = this.elementRef.nativeElement.querySelectorAll('ul');

  // Each ul element needs to be 100 / nb batches wide.
  for (let i = 0; i < ulElements.length; i++) {
    ulElements[i].style.width = 100 / this.batches.length + '%';
  }
}
```

setBatchSize sets the width of each `ul` elements. Side by side the ul elements need to take the full width of the carousel.

### slide

```typescript
slide = (direction: number) => {
  this.currentItem = this.currentItem - direction;
  this.translation  = this.translation + direction * this.increment;
  this.carousel.style.transform = 'translateX(' + this.translation + '%)';
}
```

The slide method is called each time we click on an arrow. It is used to translate from one batch to the other.

### chunk

```typescript
chunk = (arr, n) => {
  return arr.slice(0, (arr.length + n - 1) / n | 0).map((c, i) => arr.slice(n * i , n * i + n));
}
```

chunk is a simple method to split an array in an array of arrays of a specific size.

### Styling and component usage

To finish find bellow how to use the carousel and the styling file I am using.

```typescript
// app.component.ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-root',
  styleUrls: ['./app.component.scss'],
  template: `
    <app-carousel [items]='widgets'></app-carousel>
  `
})
export class AppComponent implements OnInit {
  widgets: Array<object>;

  ngOnInit() {
    this.widgets = this.generateWidgets(30);
  }

  generateWidgets = (num: number) => {
    const result = [];
    for (let i = 0; i < num; i++) {
      result.push({title: 'Widget' + i});
    }
    return result;
  }
}
```

```scss
.wrapper{
  width: 100%;
  position: relative;
}

.carousel-wrapper{
  box-sizing: border-box;
  padding: 0 40px;
  overflow: hidden;
  width: 100%;
  position: relative;
}

.carousel{
  position: relative;
  overflow: hidden;
  transition-duration: 1000ms;
}

ul{
  list-style: none;
  position: relative;
  float: left;
  width: 100%
}

li{
  width: 130px;
}

.arrow-left, .arrow-right{
  position: absolute;
  top: 6px;
  bottom: 6px;
  width: 40px;
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
  transition-duration: 300ms;
  background: white;
  z-index: 2;

  &:not(.disabled):hover{
    cursor: pointer;
    background: orange;

    i{
      color: white;
    }
  }

  &.disabled{
    i{
      color: grey !important;
    }
  }

  i{
    font-size: 3.2em;
    pointer-events: none;
  }
}

.arrow-left{
  left: 0px;
}

.arrow-right{
  right: 0px;
}
```

That's it! Have a look there for the source code: [https://github.com/HoplaGeiss/hopla-carousel](https://github.com/HoplaGeiss/hopla-carousel)
