---
title: "Angular 5 - Unit Test ngOnChanges"
description: "Unit test an angular 5 component using ngOnChanges"
tags:
  - angular5
date: "2017-11-22"
featured: false
slug: angular5-unittest-ngonchanges
image: /images/ngonchanges.png
---

In this post you will learn how to unit test a component that uses `ngOnChanges`.

### Component under test
```typescript
import { Component, OnInit, OnChanges } from '@angular/core';

@Component({
  selector: 'hopla-component',
  template: `
    <div>Hopla Hopla</div>
  `
})
export class HoplaComponent implements OnInit, OnChanges {
  @Input() data: Array<object>;

  ngOnInit(){
    // do something
  }

  ngOnChanges(){
    superFunction(data)
  }

  superFunction(input) => {
    // do something with input
  }
}
```

### Unit test

```typescript
it('OnChanges: updates the form with the site value', () => {
  const superFunctionSpy = spyOn(comp, 'superFunction');
  const data = [{value: 1}, {value: 2}]

  fixture.detectChanges();
  comp.data = data;

  comp.ngOnChanges();
  expect(superFunctionSpy).toHaveBeenCalledWith(data);
});
```

The trick is first to call `fixture.detectChanges()` to trigger `ngOnInit` then change the input and call `ngOnChanges` manually.
