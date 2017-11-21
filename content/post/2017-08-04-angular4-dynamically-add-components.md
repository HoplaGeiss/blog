---
title: "Angular 4 - Dynamically add components to the DOM"
description: "How to dynamically add components to the DOM with angular 4"
tags:
  - angular4
date: "2017-08-04"
featured: false
slug: angular4-dynamically-add-component
image: /images/dynamic.jpg
---

Today we are going to talk about dynamic component and how to add them to the DOM, but first let's take a look at how we can add content to components.

## Single content

If you are looking to add a 'single bit of content' to a component, you can use `ng-content`.

In the following example we have two components. `AppComponent` calls `HoplaComponent` and passes him some content. `HoplaComponent` access this content thanks to `ng-content`.

``` typescript
template: `
  <app-hopla>
    Some content I want to add to my component.
  </app-hopla>
`
```

``` typescript
template: `
  <ng-content></ng-content>
`
```

## Multiple contents

If you would like to add multiple content to a component you can still use `ng-content`, but in a slightly different way!

All you need to do is wrap your contents in divs and call them from your other component with a `select`.

``` typescript
template: `
  <app-hopla>
    <div class='header'>My title</div>
    <div class='footer'>My footer</div>
  </app-hopla>
`
```

``` typescript
template: `
  <ng-content select='.header'></ng-content>
  <ng-content select='.footer'></ng-content>
`
```

## Dynamic content
Let's go a step further. Let's see what happens if we try to use `ng-content` and `select` with dynamic content.

``` typescript
@Component({
  selector: 'app-root',
  template: `
    <div *ngFor='let component of [0, 1]'>
      <app-hopla>
        <div class='content_0'>A</div>
        <div class='content_1'>B</div>
      </app-hopla>
    </div>
  `
})
export class AppComponent {
}
```

``` typescript
@Component({
  selector: 'app-hopla',
  template: `
    <ng-content select='".content"index'></ng-content>
  `
})
export class HoplaComponent {
  @Input() index: number;
}
```

That won't work.. `select` is not designed to work with dynamic content. We need another approach to make that work!

We are going to use [ng-dynamic-component](https://github.com/gund/ng-dynamic-component) to do the heavy lifting for us!

``` typescript
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div *ngFor='let component of components'>
      <app-hopla [type]='component.type'></app-hopla>
    </div>
  `
})
export class AppComponent {
  components = [
    { type: 'a' },
    { type: 'b' }
  ];
}
```

We are going to go a step further from the previous Example. Instead of trying to pass dynamic content, we will pass dynamic components! As you can see in this example we try to display a list of components.

``` typescript
// hopla.component.ts
import { Component, OnChanges, Input } from '@angular/core';
import { AComponent } from './a.component';
import { BComponent } from './b.component';

@Component({
  selector: 'app-hopla',
  template: `
    <ndc-dynamic [ndcDynamicComponent]='selectedComponent'></ndc-dynamic>
  `
})
export class HoplaComponent implements OnChanges {
  @Input() type: string;

  selectedComponent: any;

  ngOnChanges() {
    if (this.type) {
      if (this.type === 'a') this.selectedComponent = AComponent;
      if (this.type === 'b') this.selectedComponent = BComponent;
    }
  }
}
```

That is where the magic happens. Depending on the type input, we display a different component!


``` typescript
// app.module.ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { DynamicModule } from 'ng-dynamic-component';

import { AComponent } from './a.component';
import { BComponent } from './b.component';
import { HoplaComponent } from './hopla.component';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent,
    AComponent,
    BComponent,
    HoplaComponent
  ],
  imports: [
    BrowserModule,
    DynamicModule.withComponents([ AComponent, BComponent ])
  ],
  bootstrap: [ AppComponent ]
})
export class AppModule { }
```

Note in the module we need to load all the possible components, so that `DynamicModule` can dynamically load is inside `HoplaComponent`.
