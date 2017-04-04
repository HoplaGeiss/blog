---
title: "Unit test $http.get"
description: "Step by step guide of how to unit test $http.get"
tags: [ "jasmine", "unit test", "angular 1.4", "$http.get",]
lastmod: 2017-01-09
date: "2017-01-09"
categories:
  - "Development"
  - "VIM"
slug: "unit-test-http-get"
image: "/images/rafale.jpeg"
---

### Scenario

In this example I will demonstrate how to unit `$http.get`. 

This is a real life example of an application, calling the backend with `$http.get` to retrieve the count of notifications.
The difficulty here is, that when our call gets resolved we don't assign a value to our scope, we just return the result. So we need to find a way to access this.

```javascript
getUnreadNotificationsCount: function (employeeId) {
  return $http.get(configuration.getFrontEndUrl('/notifications/unseen/count.json?employee_id=' + employeeId))
      .then(function (result) {
        return result.data[0];
      });
}
```

There are two assertions we need to check:
* A call has been made to `configuration.getFrontEndUrl` with the correct parameter
* The function returns the data from the `$http.get`

### Solution

```javascript
describe('getUnreadNotificationsCount function', function () {
  it('is defined', function () {
    expect(notificationApi.getUnreadNotificationsCount).toBeDefined();
  });

  it('makes a call to the backend', function () {
    spyOn(configuration, 'getFrontEndUrl').and.returnValue('fake-api-call');

    $httpBackend.when('GET', 'fake-api-call')
      .respond(200, [4]);

    notificationApi.getUnreadNotificationsCount(2).then(function (result) {
      expect(result).toEqual(4);
      expect(configuration.getFrontEndUrl).toHaveBeenCalledWith('/notifications/unseen/count.json?employee_id=2');
    });

    $httpBackend.flush();
  });
});
```

### Analysis

* We spy on `configuration.getFrontEndUrl` and return fake-api-call
* We use `$httpBackend` to mock `$http.get` and return `[4]`
* We do our assertions inside the `then` block


