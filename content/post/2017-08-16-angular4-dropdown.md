---
title: "Angular 4 dropdown"
description: "Create a simple dropdown with angular 4 without relying on bootstrap"
tags:
  - angular4
  - dropdown
date: "2017-08-16"
featured: true
slug: angular4-dropdown
image: /images/dropdown.jpg
---

Dropdowns are a very common way of displaying multiple choice inputs. Today's post is about creating a dropdown without using bootstrap or other styling frameworks.

If you are in a hurry, you can checkout the code on my github account: [https://github.com/HoplaGeiss/hopla-dropdown](https://github.com/HoplaGeiss/hopla-dropdown)

Let's first take a look at the main component.

``` typescript
// app.component.ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-root',
  styleUrls: ['./app.component.scss'],
  template: `
    <div class='wrapper'>
      <div class='selected-item' fxLayout="row" fxLayoutAlign="space-between center"
        [ngClass]='{"active": dropdownActive}'>
        <div>{{selectedItem}}</div>
        <i class='material-icons' (click)='toggleDropdown()'>keyboard_arrow_down</i>

        <div class='dropdown' [hidden]='!dropdownActive'>
          <div *ngFor='let item of items | dropdownPipe:selectedItem; let last = last'
            class='item' [ngClass]='{"last": last}' (click)='selectItem(item)'>
            <span>{{item}}</span>
          </div>
        </div>
      </div>
    </div>
  `
})
export class AppComponent implements OnInit {
  items: Array<string>;
  selectedItem: string;
  dropdownActive: boolean;

  ngOnInit() {
    this.dropdownActive = true;
    this.items = ['item 1', 'item 2', 'item 3', 'item 4'];
    this.selectedItem = this.items[0];
  }

  toggleDropdown = () => {
    this.dropdownActive = !this.dropdownActive;
  }

  selectItem = (item: any) => {
    this.selectedItem = item;
    this.dropdownActive = false;
  }
}
```

I said we would look at any easy dropdown, but actually there is quite a bit going on here. Let's go through it step by step.

We have three attributes:

-  `items` is a list of items to display. (here they are strings, but it would probably be something more complicated in real life)
- `selectedItem` is an item from `items` which is currently selected.
- `dropdownActive` a boolean that drives the state of the dropdown.

We have 2 methods:

- `toggleDropdown` toggles the value of `dropdownActive` to open and close the dropdown.
- `selectItem` changes the value of `selectedItem` to select a new item and sets `dropdownActive` to false to close the dropdown.

On initialisation we:

- Give some value to `items`.
- Set `selectedItem` to be the first item of the dropdown.
- Set `dropdownActive` to true so the dropdown is open on init.

Now let's take a look at the template. There are two main divs:

- `selected-item` which is containing our `selectedItem`.
- `dropdown` which is a list of all the other items.

The most important thing to take note of are:

- The `*ngFor` which displays the list of items.
- The `hidden` attribute on `dropdown`. That is how to show and hide the dropdown.
- The click event on the image. That is what calls the `toggleDropdown` method.
- The click event on the `item` divs. That is what calls the `selectItem` method.

Now there are also some little details to notice:

- We add an `active` class on `selected-item` when `dropdownActive` is true. That is for styling purpose. (i.e change the color of the div when the dropdown is open)
- We added a pipe on the `items` of the `*ngFor`, that is to filter the list of items to remove the selected item.
- We use last in the `*ngFor`, that is then used to add a `last` class with `ngClass` and then finally use later on for styling (remove the bottom border on the last item).

Alright, that's all for the components, let's have a quick look at the pipe for those interested.


``` typescript
// dropdown/dropdown.pipe.ts

import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'dropdownPipe'
})
export class DropdownPipe implements PipeTransform {
    transform(items: any, selectedItem: any): Array<any> {
      if (!items || !selectedItem) return null;

      return items.filter(item => {
        if (item !== selectedItem)
          return item;
      });
    }
}
```

Nothing crazy here, we just filter our the selected item from the items.

Getting the styling right is arguably the hardest when doing a dropdown. Here is a modest attempt to create a scss file.

``` scss
// app.component.scss

.wrapper{
  position: relative;
  width: 200px;
}

.selected-item{
  background: grey;
  height: 40px;
  padding: 0 10px;

  i{
    cursor: pointer;
    user-select: none;
  }

  &.active{
    background: blue;
    color: white;
    i{
      transform: rotate(180deg);
    }
  }
}

.dropdown{
  position: absolute;
  top: 40px;
  width: calc(100% - 2px);
  left: 0;
  color: black;
  border: 1px solid grey;

  .item{
    border-bottom: 1px solid grey;
    padding: 5px 10px;
    cursor: pointer;
    &:hover{
      background: grey;
    }

    &.last{
      border-bottom: 1px solid white;
      &:hover{
        border-bottom: 1px solid grey;
      }
    }
  }

}
```

Little details worth noticing:

- I added a rotation on the carret. This way it rotates as the user open and closes the dropdown.
- The dropdown is positioned absolutely.
- The last item of the list needs special care to hide its border bottom.

Again, you can check this out here: [https://github.com/HoplaGeiss/hopla-dropdown](https://github.com/HoplaGeiss/hopla-dropdown)
