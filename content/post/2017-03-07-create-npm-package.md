---
title: "Angular2 - Create a npm package"
description: "How to make a npm package out of your angular2 application"
tags: [ "angular2", "npm"]
lastmod: 2017-03-07
date: "2017-03-07"
categories:
  - "Development"
slug: "create-npm-package"
image: "/images/npm.png"
---

Creating a npm package out of your application can prove very useful. Whether you want to reuse a component you created in another of your application or if you just want to share your work with the opensource community, npm is the way to go. In this post I will describe how to create an npm package out of you angular 2 application. (created with angular cli)

You need to make sure your `package.json` isn't configured for a private application.

- Remove `private: true`
- Make sure the name of your package doesn't begin by "@something/". That is a name reserved for private packages. A valid name is `name: my-package`.

Next you need to make sure to expose the module of your application in an `index.ts` file at the root of your application

```typescript
// index.ts
export * from './src/app/app.module';
```

Also make sure to change the name of the export in the module file. This will be the name of the module to be imported.

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';

@NgModule({
  imports: [
    BrowserModule,
  ],
  declarations: [
    AppComponent
  ],
  bootstrap: [AppComponent],
  exports: [
    AppComponent
  ]
})
export class MyAwesomeModule { }
```

Then deploying your app on npm is as easy as running `npm publish`

Finally you can check if all worked properly by creating another app and installing your new package through npm!

`npm install package-name --save`

To import the package you simply need to do

`import { MyAwesomeModule } from 'package-name';`

And here you go! You are using the package you just created.
