---
title: "Unit test - Mock promises"
description: "Step by step guide of how to mock promises"
tags: [ "jasmine", "unit test", "angular 1.4", "mock", "promises"]
lastmod: 2016-12-23
date: "2016-12-23"
categories:
  - "Development"
  - "VIM"
slug: "unit-test-mock-promises"
image: "/images/promise.jpg"
---

When first writing Javascript and specially when unit testing it, promises can be one of the hardest thing to understand.

### Scenario

In this example we would like to mock a simple service returning a promise. 

This is a real life example, of an application, calling an `employeeService` to retrieve the `notificationCount`.

```javascript
employeeService.getEmployee().then(function (employee) {
  $scope.unreadNotifications = employee.notificationCount;
});
```

Usually to mock a function, we would just do:

```javascript
employeeService.getEmployee = function () {
  return {
    notification: 2
  }
}
```

However that wouldn't work here since `employeeService.getEmployee` returns a promise.

### Solution

```javascript
employeeService.getEmployee = function () {
  var deferred = $q.defer();
  deferred.resolve({
    notificationCount: 2
  });
  return deferred.promise;
};
```

### Analysis

* First we create a deferred instance `$q.defer()`
* Then we resolve it passing the object we want `{notificationCount: 2}`
* Finally we return a promise `deferred.promise`



