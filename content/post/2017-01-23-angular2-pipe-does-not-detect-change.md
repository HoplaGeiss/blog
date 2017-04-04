---
title: "Angular2 pipe - Force pipes to detect changes"
description: "Force stateless pipes to detect changes in input"
tags: [ "angular2", "pipe"]
lastmod: 2017-01-23
date: "2017-01-23"
categories:
  - "Development"
slug: "angular2-pipe-does-not-detect-change"
image: "/images/pipe.jpg"
---

The goal of this post, is to explore how pipe work and to make them do what we expect of them! Before futher due, let's see an example.
The example I am going to use, is a list of trainers filtered by sports, this list is driven by an input field.

### Scenario

```typescript
<md-card *ngFor="let trainer of trainers | criteria:'sport':sportSearch">
  {{ trainer.name }}
  {{ trainer.sport }}
</md-card>
```

The behavior you would expect, is that every time you modify the list of `sportSearch`, the list of trainers gets updated. However, when a trainer 
is added, the array reference doesn't change hence the stateless pipe is not executed.

By default pipes are stateless, meaning they don't remember anything, they just transform input data in output data.

### Solution

There are two solutions, we can either make our pipe Stateful or make sure the reference to `sportSearch` changes every time.
The second solution feels a little bit like a hack, but it is a lot more efficient then the first one. 
A stateful pipe will evaluate the input at each cycle, which makes it computational heavy.

##### Stateful pipe

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'criteria',
  pure: false
})
```

##### Changes reference
To change the value of `sportSearch`, we execute this: `this.sportSearch = search`. But that doesn't work because it doesn't change
the reference of `sportSearch`.

One way to change the reference of `sportSearch`, is to use `slice` like so: 
```typescript
this.sportSearch = search.slice()
```
Since we did not give any parameter to slice, it doesn't have any effect on the content of `sportSearch`, but it is creating a new array every time.
