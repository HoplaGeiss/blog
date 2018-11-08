---
title: "Ngrx Guards and Resolvers"
description: "When using ngrx, why is everyone implementing guards to load data instead of resolvers?"
tags:
  - angular5
  - ngrx
  - guards
  - resolvers
date: "2018-11-08"
featured: true
slug: ngrx-resolvers-guards
image: /images/data.png
---

In this post, we will have a look at an example of an ngrx application using guards and resolvers. Based on this, we will see when guards and resolvers are best using with ngrx.

Personally, I find one of the main advantages of using ngrx is to handle data in a more efficient way, e.g. not re-loading the data when it's already in state.

All the example I found online while learning about ngrx were using guards, and I didn't quite understand why that was. When you look at the documentation of guards and resolver on angular's doc, it clearly states that:
-  Guards should be used to block the navigation if certain conditions are not met; think protected route like the transition between a landing page and logged-in page or viewer mode and admin mode.
-  Resolvers should be used to load data before a page is initialized so that the data is available for the user to use.

So after reading that I went ahead and tried to implement my next feature using a resolver, that didn't go so well for me, you will see why in a bit. Now let's get started and implement our little ngrx application.

Our application is an online library and the data we are playing with are books and authors.
Books are loaded with a resolver and authors are loaded with a guard.

If you are in a hurry, you can check out the source code here: [https://github.com/HoplaGeiss/hopla-resolver-guard](https://github.com/HoplaGeiss/hopla-resolver-guard)

Let's dive in!

### Container displaying authors and books

```typescript
// library.component.ts
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs/Observable';

import * as fromStore from '../store';
import { Author } from './../models/author.model';
import { Book } from './../models/book.model';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'hopla-library',
  template: `
    <h1>Books:</h1>
    <h3 *ngFor='let book of (books$ | async)'>
      {{ book.title }}
    </h3>

    <h1>Authors:</h1>
    <h3 *ngFor='let author of authors'>
      {{ author.name }}
    </h3>
  `
})
export class LibraryComponent implements OnInit {
  books$: Observable<Book[]>;
  authors: Author[];

  constructor(
    private store: Store<any>,
    private route: ActivatedRoute
  ) {}

  ngOnInit() {
    this.books$ = this.store.select(fromStore.getBooksData);
    this.authors = this.route.snapshot.data.authors;
  }
}
```

### Book and author models

```typescript
// author.model.ts

export interface Author {
  name: string;
}
```

```typescript
// book.model.ts

export interface Book {
  title: string;
}
```

### Routing file

```typescript
// library.routing.ts

import { AuthorsResolver } from './resolvers/authors.resolver';
import { BooksGuard } from './guards/books.guard';
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

import { LibraryComponent } from './containers/library.component';

const routes: Routes = [
  {
    path: '',
    canActivate: [ BooksGuard ],
    resolve: { authors: AuthorsResolver },
    component: LibraryComponent
  }
];

@NgModule({
  imports: [
    RouterModule.forChild(routes)
  ],
})

export class LibraryRouting {}
```