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
Books are loaded with a gaurd and authors are loaded with a resolver.

If you are in a hurry, you can check out the source code here: [https://github.com/HoplaGeiss/hopla-resolver-guard](https://github.com/HoplaGeiss/hopla-resolver-guard)

Let's dive in!

### Routing file

Let's start off with our routing file. As you can see books are protected by a guard and authors are loaded with a resolver.

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

### Book guard

Our book guard is a classic guard using ngrx state. I am consecutively calling:
- The selector `getBooksLoaded` to see if the books are loaded.
- If they are not loaded I dispatch `LoadBooks` to load them.
- Then I have a `filter` to make sure I only go forward if the books are loaded
- Finally, I have `take` to stop the subscription.
- Also, I have a `catchError` that returns an observable of false to say the guard can't pass.

```typescript
import { Injectable } from '@angular/core';
import { CanActivate } from '@angular/router';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs/Observable';
import { of } from 'rxjs/observable/of';
import { catchError, filter, take, tap } from 'rxjs/operators';

import * as fromStore from '../store';

@Injectable()
export class BooksGuard implements CanActivate {
  constructor(
    private store: Store<any>
  ) {}

  canActivate(): Observable<boolean> {
    return this.store.select(fromStore.getBooksLoaded).pipe(
      tap(loaded => {
        if (!loaded) return this.store.dispatch(new fromStore.LoadBooks());
      }),
      filter(loaded => loaded),
      take(1),
      catchError(() => of(false))
    );
  }
}
```

### Author resolver

Since I didn't find any example of resolver using ngrx online, I pretty much created this on my own based on guards.
As you can see the first part is identical to the guard, the only difference is at the end. Instead of returning an observable of true or false I am actually returning the data.

```typescript
import { Injectable } from '@angular/core';
import { Resolve } from '@angular/router';
import { select, Store } from '@ngrx/store';
import { Observable } from 'rxjs/Observable';
import { filter, switchMap, take, tap } from 'rxjs/operators';

import * as fromStore from '../store';
import { Author } from './../models/author.model';

@Injectable()
export class AuthorsResolver implements Resolve<Author[]> {
  constructor(
    private store: Store<any>
  ) { }

  resolve(): Observable<Author[]> {
    return this.store.select(fromStore.getAuthorsLoaded).pipe(
      tap(loaded => !loaded ? this.store.dispatch(new fromStore.LoadAuthors()) : undefined ),
      filter(loaded => loaded),
      take(1),
      switchMap(() => this.store.pipe(select(fromStore.getAuthorsData), take(1)))
    );
  }
}
```

### Container displaying authors and books

To finish let's see how the books and the authors are used in my component.
The books are loaded in a classical way, by calling a selector.
However, for the authors we don't need to access the state, we can get them directly from the router.

At first, I thought that's an advantage as I don't need to import the store. However, then I realised that by doing so, I am not going to get the update to the authors from the state.
What I am doing here is loaded the authors before the page loads and passing a value to the router and that value does not update.

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

So here you go that's the trade of using resolvers instead of state:
- Semantically more correct, as you are trying to load data and not protect a route.
- Gives you access to the data directly, so you don't need to import the state in your component.
- It's a one time load. Your data does not get updated. To do so, you would need to subscribe to the state.