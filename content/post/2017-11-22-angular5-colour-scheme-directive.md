---
title: "Angular 5 - Colour Scheme directive"
description: "Create a dynamic colour scheme with angular 5"
tags:
  - angular5
  - scss
date: "2017-11-22"
featured: true
slug: angular5-colour-scheme
image: /images/colour-scheme.png
---

If you are developing a large application, chances are you already have thought of how to adapt the app's colour scheme to the user. In this post you will discover just how to do this!

### Bacic implementation - static colour scheme

The most basic implementation is to have a scss file with variables for your colours scheme.

```scss
  $primary: #57a1c6;
  $highlight: #4fc3f7;
  $accent: #ee7175;
```
However that only works if you have a single colour scheme.

### Advances implementation - dynamic colour scheme

To have a dynamic colour scheme, first you need a place to persist the user's choice. I choose to put in a Mongo collection. Then once the user is logged in I am retrieving this and using the user's colours scheme thanks to a directive.

##### Directive
```typescript
import { Directive, ElementRef, Renderer, OnInit, HostListener, Input } from '@angular/core';

// Services
import { ColourService } from './colour.service';

@Directive({
  selector: '[hoplaColour]'
})
export class ColourDirective implements OnInit {
  @Input() attribute = 'color';
  @Input() type = 'primary';

  ngOnInit() {
    this.changeColour(this.colourService.getColour(undefined, this.type));
  }

  constructor(
    private el: ElementRef,
    private render: Renderer,
    private colourService: ColourService
  ) {}

  private changeColour(colour: string) {
    this.render.setElementStyle(this.el.nativeElement, this.attribute, colour);
  }
}
```

This directive has two inputs, type which is the type of colours I want. As you can see in the styling file I posted above, I have 3 types: primary, secondary and highlight. The second input is attribute, this controls the css attribute I want to affect, this could be a border-color, background-color or just color for example.

##### Service

```typescript
import { Injectable } from '@angular/core';

// Services
import { ConfigService } from '../../config/config.service';

@Injectable()
export class ColourService {
  constructor (
    private configService: ConfigService
  ) {}

  getColour = (type: string): string => {
    const colours = this.configService.configSubject.getValue().colours;

    if (type in colours) return colours[type];

    return colours.primary;
  }
}
```

All this service does is using the colours stored in an Observable inside `configService.configSubject` and returning the colour in function of the type selected.
