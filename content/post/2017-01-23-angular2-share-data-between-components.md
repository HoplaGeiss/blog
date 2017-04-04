---
title: "Angular2 components - Share data between components"
description: "Share data between two siblings components in angular2"
tags: [ "angular2", "components"]
lastmod: 2017-01-23
date: "2017-01-23"
categories:
  - "Development"
slug: "angular2-share-data-between-components"
image: "/images/bees.jpg"
---

For my first post on angular2, I would like to analyse how to share data between components.
Sharing data between a parent component and a child component is fairly straight forward, but doing so between sibling components is a lot more work.

I am going to take a real life example of a home page with an search bar that drives a list of trainers.
Here the parent component is the `home.component.ts` and the two siblings are `trainer-list.component.ts` and `search-inputs.components.ts`

![Compnent architecture](/images/share-data-between-component-architecture.png)

The best way I found to share data between sibling components is to use a shared service with observable items to propagate changes.

#### The shared service

Our shared service has two methods, `setTags` to change the tags and `getTags` to get the tags using `asObservable`.

```typescript
// /home/shared/searched-tags.service.ts

import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { Subject } from 'rxjs/Subject';

@Injectable()
export class SearchedTagsService {
  private subject = new Subject<any>();

  setTags(message: string) {
    this.subject.next(message);
  }

  getTags(): Observable<any> {
    return this.subject.asObservable();
  }
}
```

### Event emitter

If you ever need to use an input with tags, I encourage you to take a look at [rosslavery/angular2-tag-input](https://github.com/rosslavery/angular2-tag-input).
Every time the user inputs a new tag we need to send an event to `trainer-list.component.ts` to notify it to change the filtering on the list of trainers.

```typescript
// /home/containers/search-inputs/search-inputs.component.ts
import { Component, OnInit } from '@angular/core';

import { SearchedTagsService } from '../../shared/searched-tags.service';

@Component({
  selector: 'search-inputs',
  template: `
    <div class="search-inputs">
      <rl-tag-input
        [addOnBlur]="false"
        [autocomplete]="true"
        [autocompleteItems]="sports"
        [(ngModel)]="sportSearchedTags"
        placeholder="Add a sport"
        (ngModelChange)="onChange($event)">
      </rl-tag-input>
    </div>
  `,
})
export class SearchInputsComponent implements OnInit{
  sports = ["Basketball", "Football"];
  sportSearchedTags: string[] = [];

  constructor(private searchedTagsService: SearchedTagsService) {}

  onChange(tags) {
    this.searchedTagsService.setTags(tags);
  }
}
```

We are calling `(ngModelChange)="onChange($event)"` every time the list of tags change and this calls our shared service's `setTags` method.

### Event receiver

Our `trainer-list.component.ts` then receives this event and changes the filtering of the list of trainers.

```typescript
// /home/containers/trainer-list/trainer-list.component.ts
import { Component, OnInit } from '@angular/core';
import { Subscription } from "rxjs";

import { SearchedTagsService } from '../../shared/searched-tags.service';

@Component({
  selector: 'trainer-list',
  template: `
    <div class="trainer-list">
      <md-card *ngFor="let trainer of trainers | criteria:'sport':sportSearch">
        {{ trainer.name }}
        {{ trainer.sport }}
      </md-card>
    </div>
  `,
  styleUrls: ['./trainer-list.component.scss']
})
export class TrainerListComponent implements OnInit{
  trainers = [
    {
      name: 'Leonnidas',
      sport: 'Football'
    },
    {
      name: 'Rambo',
      sport: 'Basketball'
    }
  ];
  subscription: Subscription;
  sportSearch: string[] = [];

  constructor(private searchedTagsService: SearchedTagsService) {}

  ngOnInit() {
    this.subscription = this.searchedTagsService.getTags()
      .subscribe(tags => {
        this.sportSearch = tags.slice();
      });
  }
}
```
As you can see, in the init function, the class is suscribing to the our service's `getTags` method, and gets the updated list of tags this way.
