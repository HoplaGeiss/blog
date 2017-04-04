---
title: "Angular 4 - create an accordion"
description: "Create an angular 4 accordion without any dependencies"
tags: [ "angular4", "accordion"]
date: "2017-04-03"
featured: true
slug: "create-angular4-accordion"
image: "/images/accordion.jpg"
---

After looking for a while for a simple accordion component on npm I was very surprised not to find any. So I began implementing my own.
To create the accordion I used two components `AccordionComponent` and `AccordionGroupComponent` and organised it in the usual component based architecture.

![Component architecture](/images/accordion-architecture.png)

Now let's take a look at both files.

``` typescript
import { Component, Input } from '@angular/core';
import { AccordionGroupComponent } from './accordion-group/accordion-group.component';

@Component({
  selector: 'accordion',
  styleUrls: ['./accordion.scss'],
  template: `
    <div class="accordion">
      <ng-content></ng-content>
    </div>
  `
})
export class AccordionComponent {
  groups: Array<AccordionGroupComponent> = [];

  addGroup(group: AccordionGroupComponent): void {
    this.groups.push(group);
  }

  closeOthers(openGroup: AccordionGroupComponent): void {
    this.groups.forEach((group: AccordionGroupComponent) => {
      if (group !== openGroup) {
        group.isOpen = false;
      }
    });
  }

  removeGroup(group: AccordionGroupComponent): void {
    const index = this.groups.indexOf(group);
    if (index !== -1) {
      this.groups.splice(index, 1);
    }
  }
}
```

```typescript
import { Component, OnDestroy, Input, OnChanges, SimpleChange } from '@angular/core';
import { AccordionComponent } from '../accordion.component';

@Component({
  selector: 'accordion-group',
  styleUrls: ['./accordion-group.scss'],
  template: `
    <div class="accordion-group" [ngClass]="{'closed': !isOpen}">
      <div class="panel-heading" (click)="toggleOpen()">
          <span>{{heading}}</span>
      </div>
      <div class="panel-collapse">
        <div class="panel-body">
          <ng-content></ng-content>
        </div>
      </div>
    </div>
  `
})

export class AccordionGroupComponent implements OnDestroy, OnChanges {
  @Input() heading: string;
  @Input() isOpen: boolean;

  constructor(private accordion: AccordionComponent) {
    this.accordion.addGroup(this);
  }

  ngOnDestroy() {
    this.accordion.removeGroup(this);
  }

  ngOnChanges(changes: {[propKey: string]: SimpleChange}) {
    for (const propName in changes) {
      if (changes.hasOwnProperty(propName)) {
        const changedProp = changes[propName];

        if (!changedProp.isFirstChange()) {
          this.accordion.groups[this.index + 1].toggleOpen();
        }
      }
    }
  }

  toggleOpen(): void {
    if (!this.isOpen) {
      this.isOpen = true;
      this.accordion.closeOthers(this);
    }
  }
}
```

As you can see most of the logic happens in `AccordionGroupComponent`, its constructor calls `AccordionComponent`'s `addGroup` method to initialise the component. Each group has a `isOpen` attribute handling its state (open / closed) and this is being watched with a `ngOnChanges` method. The actual toggling of submenues is performed by `toggleOpen` which in turn calls `accordion`'s `closeOthers` method.

To finish here is an example of how to use the accordion:

```typescript
<accordion>
  <accordion-group heading="Heading 1" [isOpen]=true>
    <span>Content 1</span>
  </accordion-group>

  <accordion-group heading="Heading 2" [isOpen]=true>
    <span>Content 2</span>
  </accordion-group>

  <accordion-group heading="Heading 3" [isOpen]=true>
    <span>Content 3</span>
  </accordion-group>
</accordion>
```

Also you can add some transition on the css to make it more smooth like so:

```scss
.accordion-group{
  position: relative;

  .panel-collapse{  
    transition-property: all;
    transition: .5s ease;

    overflow-y: hidden;
    max-height: 1000px;
  }

  &.closed{
    .panel-collapse{
      max-height: 0;
    }
  }
}
```
