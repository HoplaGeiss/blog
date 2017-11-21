---
title: "Angular 4 - Nested Routing"
description: "How to create a application with nested routing in Angular 4"
tags:
  - angular4
date: "2017-07-26"
featured: false
slug: angular4-nested-routing
image: /images/spiderweb.jpg
---

Applications are often separated in two parts. A home with the description of the application, the features, a pricing table etc.. And a second part that requires the users to log in with for example an admin panel, an overview and dashboards.

![Website Site Map](/images/nestedRouting.png)

Following the DRY principle, you should aim at having a template for the home and the main part of your website. The best way to achieve this is to have nested routes. Let me show you how I implemented this.

# AppComponent

First we need a basic root component with a `router-outlet`.

``` typescript
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <router-outlet></router-outlet>
  `
})
export class AppComponent {}
```

Then we need to decide the URL our user will use to use our app. Since users spend most of their time logged in the app, I decided to have a bare URL for `MainComponent` and `home` for `HomeComponent`.

This means the URLs will look like this:


 Home | url
--- | ---
descriptions | www.mywebsite.co.uk/home
features | www.mywebiste.co.uk/home/features
pricing | www.mywebiste.co.uk/home/pricing

Main | url
--- | ---
overview | www.mywebsite.co.uk
admin panel | www.mywebsite.co.uk/admin
dashboards | www.mywebsite.co.uk/dashboards


``` typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

// Components
import { LandingComponent } from './landing/landing.component';
import { MainComponent } from './main/main.component';

const routes: Routes = [
  { path: 'home', component: LandingComponent },
  { path: '', component: MainComponent },
];

@NgModule({
  imports: [ RouterModule.forRoot(routes) ],
  exports: [ RouterModule ]
})

export class AppRoutingModule {}
```

# MainComponent

Very similar component as for AppComponent..

``` typescript
// main/main.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-main',
  template: `
    <router-outlet></router-outlet>
  `
})
export class MainComponent {}
```

Fairly simple routing, we just need to import the different components and list them in children routes.

``` typescript
// main/main-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

// Components
import { OverviewComponent } from './overview/overview.component';
import { MainComponent } from './main.component';
import { DashboardsCompoennt } from './dashboards/dashboards.component';
import { AdminPanelComponent } from './adminPanel/adminPanel.component';

const mainRoutes: Routes = [
  {
    path: '',
    component: MainComponent,
    children: [
      { path: '', component: OverviewComponent },
      { path: 'admin', component: AdminPanelComponent },
      { path: 'dashboards', component: DashboardsCompoennt },
    ]
  }
];

@NgModule({
  imports: [
    RouterModule.forChild(mainRoutes)
  ],
  exports: [
    RouterModule
  ]
})

export class MainRoutingModule { }
```

# HomeComponent

No comment needed here. It is exactly the same as for MainComponent.

``` typescript
// home/home.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-home',
  template: `
    <router-outlet></router-outlet>
  `
})
export class HomeComponent {}
```

``` typescript
// home/home-routing.component.ts
import { NgModule } from '@angular/core';

// Routing
import { RouterModule, Routes } from '@angular/router';

// Components
import { DescriptionComponent } from './description/description.component';
import { FeaturesComponent } from './features/features.component';
import { HomeComponent } from './home.component';
import { PricingComponent } from './pricing/pricing.component';

const homeRoutes: Routes = [
  {
    path: 'home',
    component: HomeComponent,
    children: [
      { path: '', component: DescriptionComponent },
      { path: 'features', component: FeaturesComponent },
      { path: 'pricing', component: PricingComponent }
    ]
  }
];

@NgModule({
  imports: [
    RouterModule.forChild(homeRoutes)
  ],
  exports: [
    RouterModule
  ]
})

export class HomeRoutingModule { }
```
