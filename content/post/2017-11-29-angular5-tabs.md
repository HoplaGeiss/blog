---
title: "Angular 5 - Tabs"
description: "Create a tab component with angular 5"
tags:
  - angular5
date: "2017-11-29"
featured: false
slug: angular5-tabs
image: /images/tabs.jpg
---

In this post you will learn how to create a tab component with the following features:

- Resize of the tab on window resize and on tab addition / deletion.
- Two modes: deletion of tabs allowed / not allowed.
- The text in the tab needs to truncate according to the tab's size.
- The active tab needs to be highlighted.

Take a look at the demo:
[http://www.muller.tech/hopla-tabs/](http://www.muller.tech/hopla-tabs/)

If you are in a hurry, you can checkout the source code here: [https://github.com/HoplaGeiss/hopla-tabs](https://github.com/HoplaGeiss/hopla-tabs)

Let's dive in!

### Tabs container component

```scss
.wrapper{
  border-bottom: 1px solid grey;
  width: 100%;
  display: block;
  overflow: hidden;
}
```

```typescript
// ./tab/tab.component.ts
import { Component, Input, AfterViewChecked, OnInit, ElementRef, ChangeDetectorRef, OnChanges } from '@angular/core';

@Component({
  selector: 'app-tabs',
  styleUrls: ['./tabs.component.scss'],
  template: `
    <div class='wrapper' fxLayout='row' fxLayoutAlign='start stretch'>
      <div *ngFor='let tab of tabs; let i = index' (click)='selectTab(i)'>
        <app-tab [tab]='tab' [active]='i === activeTab' (closeEvent)='closeTab(i)'
          [closeAllowed]='closeTabsAllowed' [width]='tabsWidth'>
        </app-tab>
      </div>
    </div>
  `
})
export class TabsComponent implements OnInit, AfterViewChecked, OnChanges {
  @Input() tabs: Array<any>;
  @Input() closeTabsAllowed: boolean;

  activeTab = 0;
  wrapper: any;
  wrapperWidth: number;
  tabsWidth: number;
  minimumTabs = 8;

  constructor (
    private elementRef: ElementRef,
    private changeDetectorRef: ChangeDetectorRef
  ) {}

  ngOnInit() {
    this.wrapper = this.elementRef.nativeElement.querySelector('.wrapper');
  }

  // On resize, recalculate the size of tabs
  ngAfterViewChecked() {
    if (this.wrapper.offsetWidth !== 0 && this.wrapper.offsetWidth !== this.wrapperWidth) {
      this.wrapperWidth = this.wrapper.offsetWidth;

      this.setTabSize();
    }
  }

  ngOnChanges() {
    if (this.tabs.length > 0) this.setTabSize();
  }

  selectTab = (index: number) => {
    this.activeTab = index;
  }

  closeTab = (index: number) => {
    this.tabs.splice(index, 1); // Remove the tab.

    // Change in activeTab
    if (index === this.activeTab) this.activeTab = 0;
    if (index < this.activeTab) this.activeTab--;

    this.setTabSize(); // Resize the tabs
  }

  setTabSize = () => {
    const numberTab = this.tabs.length > this.minimumTabs ? this.tabs.length : this.minimumTabs;
    this.tabsWidth = this.wrapperWidth / numberTab;
    this.changeDetectorRef.detectChanges();
  }
}
```

A couple things there:

- `setTabSize` is the method used to.. set the width of the tabs! It is called on resize, on tab deletion and on tab addition. By default we base the width of the tabs on 8 being displayed full width on the screen, if we have more then 8 tabs, the tabs resize.
- `closeTab` is a little tricky. You need to think of changing the index of the selected tab according to the position of the deleted tab.

You will notice the template makes use of the `app-tab` component. Let's have a look at this one now!

### Tab component

```scss
.tab{
  box-sizing: border-box;
  position: relative;
  border-bottom: 2px solid white;
  margin-bottom: 1px;
  padding: 0 10px;

  .text{
    width: 50px;
    white-space: nowrap;
    overflow: hidden;
  }

  &.active{
    border-bottom-color: #ee7175;
  }

  &:not(.active):hover{
    cursor: pointer;
    border-bottom-color: #d2d7d3;
  }

  &:hover{
    .close-icon{
      opacity: 1;
    }
  }

  .close-icon{
    z-index: 2;
    position: absolute;
    right: 0px;
    top: 0px;
    font-size: 1em;
    opacity: 0;
    cursor: pointer;
    color: #d2d7d3;

    &:hover{
      color: #57a1c6;
    }
  }
}
```

```typescript
import { Component, Input, Output, EventEmitter, OnChanges, ElementRef, OnInit } from '@angular/core';

@Component({
  selector: 'app-tab',
  styleUrls: ['./tab.component.scss'],
  template: `
    <div class='tab' [ngClass]='{"active": active}'>
      <div class='text'>{{tab.name}}</div>
      <i *ngIf='closeAllowed' class='material-icons close-icon' (click)='close($event)'>clear</i>
    </div>
  `
})
export class TabComponent implements OnChanges {
  @Input() tab: any;
  @Input() active: boolean;
  @Input() closeAllowed: boolean;
  @Input() width: number;
  @Output() closeEvent: EventEmitter<any> = new EventEmitter();

  constructor (
    private elementRef: ElementRef,
  ) {}

  ngOnChanges() {
    const tabElement = this.elementRef.nativeElement.querySelector('.tab');
    const textElement = this.elementRef.nativeElement.querySelector('.text');

    if (tabElement) {
      tabElement.style.width = this.width + 'px';
      textElement.style.width = closeAllowed ? this.width - 28 + 'px' : this.width - 5 + 'px';
      tabElement.style.transition = 'width 400ms';
    }
  }

  close = (event) => {
    event.stopPropagation();
    this.closeEvent.emit();
  }
}
```

Here again some things to take note of:

- Based on `closeAllowed`, we display the little cross to close the tab or not.
- the `close` method uses `event.stopPropagation()`, that is important otherwise the click is picked up by the tab itself. ( the tab would become deleted and selected!)
- In the `ngOnChanges` updates the width of the tab and of the element inside. You will notice that when `closeAllowed` is true, the text element inside is a little smaller then the tab, that is to be able to fit the delete cross.

To finish, find bellow an example of how this component can be used!

### Using our tabs

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  styleUrls: ['./app.component.scss'],
  template: `
    <app-tabs [tabs]='tabs' [closeTabsAllowed]='true'></app-tabs>

    <button (click)='addTab()'>Add Tab</button>
  `
})
export class AppComponent {
  tabs = [
    { name: 'Super long tab' },
    { name: 'Tab2' },
    { name: 'Tab3' },
    { name: 'Tab4' },
    { name: 'Tab5' },
    { name: 'Tab6' },
    { name: 'Tab7' },
  ];

  index = 8;

  addTab = () => {
    this.tabs.push({ name: 'Tab' + this.index });
    this.index++;
    this.tabs = this.tabs.slice(); // hack for change detection;
  }
}
```

Have a look here to see the source code: [https://github.com/HoplaGeiss/hopla-tabs](https://github.com/HoplaGeiss/hopla-tabs)
