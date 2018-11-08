---
title: "Angular 5 - Search directive to filter a list"
description: "Create search directive filtering a dynamic list with angular 5"
tags:
  - angular5
date: "2017-11-30"
featured: false
slug: angular5-search-filter
image: /images/search.png
---

In this post you will learn how to create a search directive filtering a dynamic list with the following features:

- The directive will have as input a list of items, the output will be a filtered list of items.
- The directive needs to be able to react to a change in the input.

Take a look at the demo:
[http://www.muller.tech/hopla-search/](http://www.muller.tech/hopla-search/)

If you are in a hurry, you can checkout the source code here: [https://github.com/HoplaGeiss/hopla-search](https://github.com/HoplaGeiss/hopla-search)

Let's dive in!

### Pipe filtering

First thing first, we are going to implement a pipe to filter our list.

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'SearchPipe'
})
export class SearchPipe implements PipeTransform {
  transform(items: Array<any>, attribute: string, term: string): Array<any> {
    if (!term || term === '') return items;

    return items.filter(item => {
      if (item[attribute].toLowerCase().startsWith(term.toLowerCase())) return item;
    });
  }
}
```

This pipe takes 3 elements:

- The array of items to filter.
- The attribute of the item used to do the filtering.
- The search term used to do the filtering.

### Directive

Now all we need is to create a directive that glues the piece together!

```typescript
import { Directive, Input, OnChanges, EventEmitter, Output, HostListener } from '@angular/core';

import { SearchPipe } from './search.pipe';

@Directive({
  selector: '[appSearch]'
})
export class SearchDirective implements OnChanges {
  @Input() items: Array<any>;
  @Input() attribute: string;
  @Output() filterEvent: EventEmitter<any> = new EventEmitter();

  searchTerm: string;

  @HostListener('keyup', ['$event.target.value']) onKeyUp(value) {
    this.searchTerm = value;
    this.applyFilter();
  }

  ngOnChanges() {
    this.applyFilter();
  }

  constructor (
    private searchPipe: SearchPipe
  ) {}

  applyFilter = () => {
    this.filterEvent.emit(new SearchPipe().transform(this.items, this.attribute, this.searchTerm));
  }
}
```

So in this directive the logic happens in the `applyFilter`, which is just a call to our pipe. The `applyFilter` method gets called either when items change (ngOnChanges) or when the search term changes (a key is pressed).


### How to use the directive

As usual let's finish the post by showing you how to use what we just created!

```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-root',
  styleUrls: ['./app.component.scss'],
  template: `
    <input appSearch (filterEvent)='filterUsers($event)' [items]='users' [attribute]='"name"'>
    <div class='users-list'>
      <div *ngFor='let user of filteredUsers'>{{user.name}}</div>
    </div>
  `
})
export class AppComponent implements OnInit {
  usersList = [
    { name: 'Aa' },
    { name: 'Ab' },
    { name: 'Ac' },
    { name: 'Ba' },
    { name: 'Bc' },
    { name: 'Bd' },
    { name: 'Ca' },
    { name: 'Da' },
  ];
  users = this.usersList.slice();
  filteredUsers = this.usersList.slice(); // copy of users

  ngOnInit() {
    setInterval(() => {
      this.users = this.usersList.slice(0, Math.floor(Math.random() * this.usersList.length));
    }, 2000);
  }

  filterUsers = (users) => {
    this.filteredUsers = users;
  }
}
```

That's it! Have a look there for the source code: [https://github.com/HoplaGeiss/hopla-search](https://github.com/HoplaGeiss/hopla-search)
