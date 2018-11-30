---
title: "Angular 5 - Input directive"
description: "Create an input directive that highlights errors with angular 5"
tags:
  - angular5
date: "2017-12-01"
featured: false
slug: angular5-input-directive
image: /images/input.png
draft: true
---

In this post you will learn how to create an input directive that highlights errors.

Take a look at the demo:
[http://www.muller.tech/hopla-input/](http://www.muller.tech/hopla-input/)

If you are in a hurry, you can checkout the source code here: [https://github.com/HoplaGeiss/hopla-input](https://github.com/HoplaGeiss/hopla-input)

Let's dive in!

### Input directive

```typescript
import { Directive, Input, HostBinding, OnChanges } from "@angular/core";

import { ColourService } from "../colour/colour.service";

@Directive({
  selector: "[appInput]"
})
export class InputDirective implements OnChanges {
  @Input() error: boolean;

  @HostBinding("style.border-bottom-color") borderColor: string;
  @HostBinding("style.border-width") borderWidth = "0 0 2px 0";
  @HostBinding("style.outline") outline = "none";

  constructor(private colourService: ColourService) {}

  ngOnChanges() {
    const colourType = this.error ? "accent" : "primary";
    this.borderColor = this.colourService.getColour(colourType);
  }
}
```

3 things here:

- I am using `@HostBinding` to change the styling of the input.
- I have an error `@Input` to notify the directive it needs to change it state.
- I am pulling the colours from an external service `colourService`.

Let's have a look at this colourService

### Colour service

```typescript
import { Injectable } from "@angular/core";

@Injectable()
export class ColourService {
  getColour = (type: string): string => {
    if (type === "accent") return "red";

    return "blue";
  };
}
```

Nothing crazy here, I return red in case there is an error and blue otherwise. However you could have a much more complicated service. For example this service could call another service that is in charge of getting the configuration of the app per tenant. This would be for instance the case if you had a different colour scheme per tenant.

To finish let's see how to use the input directive!

### Usage of our new input directive

```typescript
import { Component } from "@angular/core";
import { FormControl } from "@angular/forms";

@Component({
  selector: "app-root",
  styleUrls: ["./app.component.scss"],
  template: `
    <input
      appInput
      required
      [formControl]="termSearch"
      [error]="hasErrors()"
      required
    />
  `
})
export class AppComponent {
  termSearch = new FormControl();

  hasErrors = (): boolean => {
    return this.termSearch.invalid && this.termSearch.dirty;
  };
}
```

Here I am using the `hasError` method to define the state of the `error` input. In our case we define an error to be displayed when the input has been touched and is empty.

That's it! Have a look there for the source code: [https://github.com/HoplaGeiss/hopla-input](https://github.com/HoplaGeiss/hopla-input)
