---
title: "Unit test events"
description: "Step by step guide of how to mock promises"
tags: [ "jasmine", "unit test", "angular 1.4", "events"]
lastmod: 2017-01-09
date: "2017-01-09"
categories:
  - "Development"
  - "VIM"
slug: "unit-test-events"
image: "/images/events.jpg"
---


### Scenario

In this example I will demonstrate how to unit test an event. 
The difficulty here, is to set the element's target.


```javascript
$scope.toggleRadioButton = function (e) {
  e.stopImmediatePropagation();
  if (angular.element(e.target).hasClass('my-radio-button')) {
    $scope.selectedButton = true;
  }
};
```

### Solution

```javascript
it('stopImmediatePropagation is called and $scope.selectedButton is set to true', function () {
  elem = angular.element('<div class="my-radio-button"></div>');

  event = {
    target: elem,
    stopImmediatePropagation: function () {}
  };

  spyOn(event, 'stopImmediatePropagation');
  scope.toggleRadioButton(event);

  expect(event.stopImmediatePropagation).toHaveBeenCalled();
  expect(scope.selectedButton).toEqual(true);
});
```

### Analysis

* We create an element with the class `my-radio-button`
* We create an event with two parameters, the element we just created and a fake stopImmediatePropagation function
* We spy on event.stopImmediatePropagation



