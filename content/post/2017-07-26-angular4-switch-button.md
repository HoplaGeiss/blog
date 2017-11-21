---
title: "Angular 4 - Switch button"
description: "Create a simple stylish switch button with angular 4"
tags:
  - angular4
date: "2017-07-26"
featured: false
slug: angular4-switch-button
image: /images/switchButton.jpg
---

Ever wondered how to create a switch button in angular4 without using css frameworks? Here we go..

Let's take a look at the component first.

``` typescript
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div class="switch">
      <input id='switch-input' type='checkbox' [checked]='switchActive' (click)='toggle()'>
      <label for='switch-input'></label>
    </div>
  `
})
export class AppComponent {
  switchActive = false

  toggle = () => {
    if (this.switchActive) {
      // Do something
    } else {
      // Do something else
    }

    this.switchActive = !this.switchActive;
  }
}
```

As you can see the component is base on a simple checkbox element and a toggle function to change the state of this component. Let's put some styling on top of that..

``` scss
/deep/ .switch{
  input {
    position: absolute;
    margin-left: -9999px;
    visibility: hidden;
  }

  label {
    display: block;
    position: relative;
    cursor: pointer;
    outline: none;
    user-select: none;
    width: 46px;
    height: 24px;
    border-radius: 24px;
    transition: background 0.4s;
    border: 2px solid #dddddd;

    &:after{
      display: block;
      position: absolute;
      content: "";
      top: 2px;
      left: 2px;
      bottom: 2px;
      width: 16px;
      background-color: blue;
      border-radius: 20px;
      transition: margin 0.4s, background 0.4s;
    }
  }

  input:checked + label {
    border-color: blue;
  }

  input:checked + label:after {
    margin-left: 22px;
    background-color: blue;
  }
}
```

Note I am using `/deep/` so that I can define this styling once and from anywhere in my app instead of having to redefine it each time I am using this component. Typically I keep global styling in my `app.component.scss`.
