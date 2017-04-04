---
title: "Unit test the removal of a class"
description: "Step by step description of how to unit test the removal of a class"
tags: [ "jasmine", "unit test", "angular 1.4" ]
lastmod: 2016-12-22
date: "2016-12-22"
categories:
  - "Development"
  - "VIM"
slug: "unit-test-removal-of-a-class"
image: "/images/removal.jpg"
---

### Scenario

My app loads with a splash screen, this splash screen disappears after a timeout. I would like to unit test the disparition of the splash screen.

Splash screen:
```html
<div class="splash on"></div>
```

Code driving the disparition of the splash screen:
```javascript
$timeout(function () {
    angular
    .element('.splash')
    .removeClass('on');
  }, 2000);
```


### Solution

First we need a way to create find the element in the DOM. We need to assert the element has the close `on`, then wait for the `$timeout` to finish and finally assert again that class `on` disappeared.

```javascript
  var elem = angular.element('<div class="splash on"></div>');
  elem.appendTo('body');

  expect(elem.length).toBe(1);
  expect(elem.hasClass('on')).toEqual(true);
  $timeout.flush();
  expect(elem.hasClass('on')).toEqual(false);
```

### Analysis

* Get the element with `angular.element`
* Adds the element to the body with `appendTo('body')`.

* Check we have an element
* Check our element has a class of `on`
* Flush the `$timeout`
    * This is a very useful trick that can be used a number of situation
* Check our element doesn't have the class `on` anymore
